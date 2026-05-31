# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An open-source, community-driven knowledge base documenting real experiences
building AI-agent workflows with open-source tools. The value is in honest
failure reports and working configs — not polished theory. Born from a real
n8n marketing automation project at Dot.ID.

This is a **pure Markdown repository** — no build system, no package manager,
no tests. All work is writing, editing, and committing `.md` files.

This CLAUDE.md follows Andrej Karpathy's pattern for AI agent operating manuals —
see [`lessons/claude-md-for-ai-agent-projects.md`](lessons/claude-md-for-ai-agent-projects.md).

---

## Repository Structure

```
.
├── CLAUDE.md                    ← this file
├── README.md                    ← public-facing overview + browsing table
├── CONTRIBUTING.md              ← how to add entries (full style guide)
├── LICENSE                      ← MIT
├── case-studies/
│   ├── TEMPLATE.md              ← canonical template for new case studies
│   ├── 001-social-media-image-publishing.md
│   ├── 002-n8n-binary-data-mode.md
│   ├── 003-ai-media-generation-stack.md
│   └── 004-rag-pgvector-whatsapp-agent.md
├── tools/
│   ├── google-drive.md
│   ├── garage-s3.md
│   ├── wordpress-media.md
│   ├── n8n-ce.md
│   └── coolify.md
└── lessons/
    ├── public-image-urls-for-social.md
    ├── n8n-binary-data.md
    └── claude-md-for-ai-agent-projects.md
```

---

## Content Architecture

### Three Content Types

**Case Studies** (`case-studies/`)
End-to-end stories structured as: goal → tools tried (in order) → failures →
working solution. Each attempt has its own "Why we tried it / How we set it up /
What happened / Verdict" block. Numbered sequentially (`001`, `002`, …).
Use `case-studies/TEMPLATE.md` as the canonical starting point.

**Tool Pages** (`tools/`)
Single-tool reference pages. Structure: what it is → why people try it →
the problem or gotcha → working config or fix → alternatives. One file per tool,
named after the tool (`n8n-ce.md`, `wordpress-media.md`, etc.).

**Lessons** (`lessons/`)
Cross-cutting principles extracted from multiple case studies. Structure: the
principle stated plainly → why it bites people → the safe pattern or fix →
verification steps. These generalize beyond any single tool.

### Cross-Linking Convention

Every entry links to related files using **relative Markdown paths**:
- From `case-studies/` → `../tools/tool-name.md` or `../lessons/lesson-name.md`
- From `tools/` → `../case-studies/NNN-slug.md` or `../lessons/lesson-name.md`
- From `lessons/` → `../case-studies/NNN-slug.md` or `../tools/tool-name.md`

Always add a `## Related` or `## See Also` section at the bottom of every entry.

---

## Key Design Decisions

### Failure-First Framing
Every case study documents the failed attempts in full detail before the working
solution. The failure section is the most valuable part — it must include the
exact error message, the symptom (especially silent failures), and what made the
problem hard to debug. "It didn't work" is not a failure report.

### Verdict Icons — Three Only
All evaluations use exactly one of these, no variations:
- `✅ Worked`
- `⚠️ Partial`
- `❌ Failed`

### Sequential Case Study Numbering
Case studies use zero-padded three-digit prefixes (`001`, `002`, …). Check the
highest existing number before creating a new file. Tool pages and lessons are
not numbered — they're named after their topic.

### TODO Stubs Over Omissions
If a section can't be filled in (e.g. an error wasn't fully captured), write a
`> TODO: describe the exact error here` block rather than leaving the section
empty or removing it. Stubs invite community contributions; gaps look like the
section was forgotten.

---

## Privacy Rules (Public Repo)

Content is often drawn from internal projects. Strip the following before committing:

| Strip | Replace with |
|---|---|
| Private server domains / IPs | `your-server.com` |
| Numeric user IDs in table names (`u42_vectors`) | Pattern form (`u{user_id}_vectors`) |
| Email addresses with product/company names | "system account" or omit |
| Client names | omit |
| Pricing figures | omit |
| Private GitHub repo URLs | omit |

**Keep:** version numbers, exact error messages, CLI commands, timing data,
architecture patterns, model IDs, API endpoint paths.

---

## Common Gotchas

### Cross-links break if slugs change
File renames break every `../tools/old-name.md` link in the repo. Grep for the
old name before renaming: `grep -r "old-name" .`

### TEMPLATE.md must stay complete
`case-studies/TEMPLATE.md` is the source of truth for the case study format.
If you change the format in existing entries, update the template to match so
new contributors follow the right structure.

### Verdict in the tool page header vs. body
The first line of every tool page sets context with a one-line verdict:
```
**Verdict:** ✅ Worked — reliable public image host for social media automation
```
This is separate from per-attempt verdicts inside case studies. Don't conflate them.

---

## Agent Behavior Guidelines

Adapted from Karpathy's four principles for agentic coding — applied here to
documentation work. Full rationale in
[`lessons/claude-md-for-ai-agent-projects.md`](lessons/claude-md-for-ai-agent-projects.md).

### 1. Think Before Writing
If the scope of a new entry is ambiguous — what experience it covers, which
content type fits, whether it overlaps an existing file — surface the ambiguity
before writing. Don't silently pick one interpretation and produce the wrong entry.

### 2. Simplicity First
Minimum content to document the experience honestly. No padding, no generic
advice that belongs in any tutorial, no sections repeated from CONTRIBUTING.md.
If a point is already covered in a linked file, link — don't duplicate.

### 3. Surgical Edits
When improving an existing entry, change only the section that needs it. Don't
reformat adjacent paragraphs, don't rewrite sections that aren't broken, don't
"clean up" style that wasn't part of the request.

### 4. Verifiable Completion
Before marking an entry done, confirm:
- Every template section is filled or has a `> TODO` stub (no silent omissions)
- Every relative cross-link points to a file that actually exists
- The verdict icon in the header matches the conclusion in the body

---

## Commit Message Convention

```
add: <short description>        # new file(s)
update: <tool or case study>    # editing existing content
fix: <what was wrong>           # typo / broken link / factual correction
```

---

## Content Flow Summary

```
Real experience (internal project)
    ↓
Sanitize (strip private details — see Privacy Rules)
    ↓
Choose content type:
    Case study → copy TEMPLATE.md → NNN-slug.md
    Tool page  → tools/tool-name.md
    Lesson     → lessons/lesson-name.md
    ↓
Fill all sections (use > TODO for gaps, don't omit)
    ↓
Add ## Related / ## See Also with relative cross-links
    ↓
Commit (add:/update:/fix: prefix) → push to master
    ↓
GitHub release (gh release create vX.Y.Z) for significant milestones
```
