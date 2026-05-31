# 004 — Building a Per-Tenant RAG Knowledge Base with n8n + pgvector

> **Status:** Complete  
> **Stack:** n8n · PostgreSQL + pgvector · OpenAI text-embedding-3-small · Laravel · WhatsApp Business API  
> **Context:** Multi-tenant WhatsApp AI agent SaaS — each client's agent answers questions grounded in their own business knowledge

---

## The Goal

Each client (a business using the platform) needs their AI agent to answer
questions accurately from their specific knowledge — product info, FAQs, pricing,
policies — without hallucinating or mixing up content between clients.

The end state: client uploads a knowledge item (text, FAQ, URL, PDF) via a
portal UI → it gets chunked, embedded, and stored → the AI agent retrieves
relevant chunks at query time → answers are grounded in the client's actual content.

Requirements:
- Fully automated ingestion pipeline (no manual embedding step)
- Strict per-tenant isolation (client A's agent must never surface client B's content)
- Runs on self-hosted infrastructure (no managed vector DB cost)
- Integrates with an existing n8n automation stack

---

## Architecture Decision: Per-Tenant Tables in pgvector

**The option we didn't take: one shared table with a tenant ID column**

```sql
-- Single-table approach (rejected)
CREATE TABLE knowledge_vectors (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  content TEXT,
  embedding VECTOR(1536),
  ...
);
```

This is simpler to set up, but it means every similarity search scans all tenants'
vectors (even with an index on `user_id`) and makes a data-isolation mistake (a
missing `WHERE user_id = ?`) catastrophic — one bad query surfaces another client's
data.

**What we did: one table per tenant**

```sql
-- Per-tenant table (one per client, created on signup)
CREATE TABLE u{user_id}_vectors (
  id SERIAL PRIMARY KEY,
  content TEXT,
  embedding VECTOR(1536),
  metadata JSONB,
  source_type VARCHAR(50),  -- 'text', 'url', 'pdf', 'faq'
  status VARCHAR(20)        -- 'pending', 'embedded', 'failed'
);

CREATE INDEX ON u{user_id}_vectors USING ivfflat (embedding vector_cosine_ops);
```

Each client's vectors live in their own table. A bug can't leak data across
tenants. Searches are scoped by table name, not by a `WHERE` clause that might
be forgotten.

**Gotcha: Postgres table names cannot start with a digit**

If your user IDs are numeric (e.g. `42`), a table named `42_vectors` causes:

```
ERROR: syntax error at or near "42"
```

Prefix with a letter: `u42_vectors`, not `42_vectors`. See also
[`tools/n8n-ce.md`](../tools/n8n-ce.md) for this same issue in Chat Memory tables.

---

## The Ingestion Pipeline (n8n Knowledge Writer Workflow)

When a client uploads a knowledge item through the portal UI, the backend creates
a `ClientKnowledgeItem` record with `status = pending` and triggers the n8n
Knowledge Writer webhook.

```
[Webhook trigger]
    → [Fetch item content from portal API]
    → [Chunk text — 1000 chars / 200 char overlap]
    → [For each chunk: OpenAI text-embedding-3-small]
    → [INSERT into u{user_id}_vectors]
    → [Update status → 'embedded' on portal API]
```

### Chunking

We use fixed-size character chunking with overlap, not sentence or paragraph
splitting. This was the simpler starting point and worked well enough for FAQ-style
content:

```js
// Code node — chunk with overlap
function chunk(text, size = 1000, overlap = 200) {
  const chunks = [];
  let start = 0;
  while (start < text.length) {
    chunks.push(text.slice(start, start + size));
    start += size - overlap;
  }
  return chunks;
}
```

For large documents (>10 sections), split the source material into multiple
knowledge items before uploading rather than sending one huge block. The
embedding API has a token limit per request, and very long chunks reduce
retrieval precision anyway.

### Embedding

```
POST https://api.openai.com/v1/embeddings
{
  "model": "text-embedding-3-small",
  "input": "chunk text here"
}
```

`text-embedding-3-small` (1536 dimensions) is the right default: cheaper and
faster than `text-embedding-3-large`, with negligible quality difference for
business FAQ content. Use `large` only if you're embedding very technical or
specialized text where subtle semantic differences matter.

### Storing in pgvector

```sql
INSERT INTO u{user_id}_vectors (content, embedding, metadata, source_type, status)
VALUES ($1, $2::vector, $3, $4, 'embedded');
```

The `::vector` cast is required — pgvector won't accept a plain float array
without it.

---

## The Retrieval Pipeline (at query time)

When the AI agent receives a WhatsApp message, it queries the client's vector
table before calling the LLM:

```sql
SELECT content, metadata,
       1 - (embedding <=> $1::vector) AS similarity
FROM u{user_id}_vectors
WHERE status = 'embedded'
ORDER BY embedding <=> $1::vector
LIMIT 5;
```

`<=>` is the cosine distance operator in pgvector. `1 - distance` gives you
cosine similarity (higher = more relevant). The top 5 chunks are injected into
the system prompt as context.

**Threshold filtering:** We don't use a hard similarity cutoff for now. If no
knowledge is uploaded, the agent falls back to its base system prompt. If very
low-quality matches are causing hallucination, add `WHERE 1 - (embedding <=> $1::vector) > 0.75`.

---

## Knowledge Source Types

The portal UI accepts four source types, each handled differently in the ingestion
pipeline:

| Type | How it's processed |
|---|---|
| `text` | Chunk directly and embed |
| `faq` | Treated as pre-chunked (one Q&A pair = one chunk), embed as-is |
| `url` | HTTP fetch → extract body text → chunk → embed |
| `pdf` | Extract text with a PDF parser → chunk → embed |

Start with `text` and `faq` — they're the most reliable. URL and PDF ingestion
add failure modes (unreachable URLs, scanned PDFs with no text layer) that
require error handling.

---

## Dedicated System Agent for Product Knowledge

We created a dedicated system-level account in the platform with its own vector
table, used to ground the customer-facing sales agent. Its knowledge base
documents the product itself — features, pricing, onboarding steps, FAQs.

This keeps product knowledge separate from any individual client's data and lets
you update the sales agent's knowledge by uploading a new document, without
touching code or prompts.

---

## What We Shipped

```
Client uploads knowledge item (portal UI)
    → ClientKnowledgeItem created, status=pending
    → n8n Knowledge Writer webhook
        → Fetch content
        → Chunk (1000/200 overlap)
        → Embed each chunk (text-embedding-3-small)
        → INSERT into u{user_id}_vectors
        → PATCH status=embedded on portal API

WhatsApp message received
    → n8n agent workflow
        → Embed incoming message (text-embedding-3-small)
        → SELECT top-5 from u{user_id}_vectors by cosine distance
        → Inject chunks into system prompt
        → LLM call (Claude/GPT) with grounded context
        → Reply via WhatsApp API
```

---

## Key Takeaways

- **Per-tenant tables beat a shared table with a tenant column** for hard isolation and query simplicity. The overhead is worth it.
- **Postgres table names can't start with a digit** — prefix numeric IDs with a letter.
- **1000/200 chunking is a safe default** for FAQ and business-description content. Tune only if retrieval quality is demonstrably poor.
- **`text-embedding-3-small` is the right starting model** — cheaper, fast, sufficient for most business content.
- **A dedicated system agent account** is the cleanest way to ground a product-facing agent without mixing its knowledge with client data.

---

## Related

- [`tools/n8n-ce.md`](../tools/n8n-ce.md) — the Postgres digit-prefix gotcha and other n8n CE quirks
- [`tools/coolify.md`](../tools/coolify.md) — how this stack is deployed
- [`case-studies/002-n8n-binary-data-mode.md`](002-n8n-binary-data-mode.md) — binary handling in the same n8n environment
