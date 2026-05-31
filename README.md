# Open-Source Library for AI Agents

> A practical, experience-driven knowledge base documenting what actually works
> (and what doesn't) when building workflows with open-source tools and AI coding
> agents like Claude Code.

## What This Is

This repository is a community knowledge base. Instead of polished theory, it
collects real war stories: the tool we tried, why we tried it, whether it worked,
and what we switched to when it didn't. The goal is to save the next person the
hours we spent debugging dead ends.

Born out of a real n8n marketing-automation project at Dot.ID, where we needed
AI-generated images to publish automatically to WordPress and social media.

## Why It Exists

When you build with AI coding agents, the agent recommends a tool, you implement
it, and sometimes it just… fails — for reasons no tutorial mentions. We want to
capture those failure modes and the working alternatives, so the open-source +
AI-agent community can move faster.

## Browsing the Knowledge Base

| Section | What's inside |
|---|---|
| [`case-studies/`](case-studies/) | End-to-end stories: goal → tools tried → what worked |
| [`tools/`](tools/) | Per-tool verdicts, gotchas, and working configs |
| [`lessons/`](lessons/) | Cross-cutting principles distilled from the case studies |

---

## Case Studies

| # | Title | Stack | Verdict |
|---|---|---|---|
| [001](case-studies/001-social-media-image-publishing.md) | Auto-publishing AI images to WordPress and social media | n8n · Google Drive · WordPress · Instagram | Google Drive ❌ → Garage S3 ❌ → WordPress Media ✅ |
| [002](case-studies/002-n8n-binary-data-mode.md) | n8n `binaryDataMode: database` trap | n8n · Code nodes · HTTP Request | Silent corruption ❌ → `helpers.getBinaryDataBuffer` ✅ |
| [003](case-studies/003-ai-media-generation-stack.md) | AI image and video generation stack | n8n · DALL-E · Runway · Gemini · Veo | DALL-E ⚠️ → Runway ⚠️ → Gemini 3 Pro + Veo 3.1 ✅ |
| [004](case-studies/004-rag-pgvector-whatsapp-agent.md) | Per-tenant RAG knowledge base | n8n · PostgreSQL · pgvector · OpenAI embeddings | Per-tenant tables + text-embedding-3-small ✅ |

---

## Tools

| Tool | Verdict | Key finding |
|---|---|---|
| [Google Drive](tools/google-drive.md) | ❌ Failed | Auth wall blocks social platform crawlers |
| [Garage S3](tools/garage-s3.md) | ⚠️ Unverified | Community TODO — config gotchas documented |
| [WordPress Media Library](tools/wordpress-media.md) | ✅ Worked | Returns a clean, truly public direct URL |
| [n8n Community Edition](tools/n8n-ce.md) | ✅ Worked | Six CE 2.15 gotchas documented (binary mode, webhook wrapping, Postgres naming, and more) |
| [Coolify](tools/coolify.md) | ✅ Worked | Safe update procedure; real beta→stable upgrade experience |

---

## Lessons

| Lesson | One-line summary |
|---|---|
| [Public image URLs for social](lessons/public-image-urls-for-social.md) | Social platforms need a truly public direct URL — Drive and Dropbox links silently fail |
| [n8n binary data gotchas](lessons/n8n-binary-data.md) | `.data` is not base64 in database mode; HTTP Request wipes binary collection on output |
| [CLAUDE.md for AI agent projects](lessons/claude-md-for-ai-agent-projects.md) | Karpathy's four principles for grounding an AI agent in your project |
| [Building a good CLAUDE.md](lessons/building-a-good-claude-md.md) | Three-pass pattern: `/init` → structural template → Karpathy's principles |

---

## Contributing

Got a tool that worked — or burned you? See [CONTRIBUTING.md](CONTRIBUTING.md)
and open a PR. Every honest failure report is worth as much as a success story.

## License

MIT — use it, fork it, share your own experiences.
