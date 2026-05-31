# Open-Source Library for AI Agents

> A practical, experience-driven knowledge base documenting what actually works
> (and what doesn't) when building marketing automation and other workflows with
> open-source tools and AI coding agents like Claude Code.

## What This Is

This repository is a community knowledge base. Instead of polished theory, it
collects real war stories: the tool we tried, why we tried it, whether it worked,
and what we switched to when it didn't. The goal is to save the next person the
hours we spent debugging dead ends.

Born out of a real n8n marketing-automation project at Dot.ID, where we needed
AI-generated images to publish automatically to WordPress AND social media.

## Why It Exists

When you build with AI coding agents, the agent recommends a tool, you implement
it, and sometimes it just… fails — for reasons no tutorial mentions. We want to
capture those failure modes and the working alternatives, so the open-source +
AI-agent community can move faster.

## Repository Structure

```
.
├── README.md
├── CONTRIBUTING.md
├── case-studies/
│   ├── TEMPLATE.md
│   └── 001-social-media-image-publishing.md
├── tools/
│   ├── google-drive.md
│   ├── garage-s3.md
│   └── wordpress-media.md
└── lessons/
    └── public-image-urls-for-social.md
```

## Browsing the Knowledge Base

| Section | What's inside |
|---|---|
| [`case-studies/`](case-studies/) | End-to-end stories: goal → tools tried → what worked |
| [`tools/`](tools/) | Per-tool verdicts, gotchas, and working configs |
| [`lessons/`](lessons/) | Cross-cutting principles distilled from the case studies |

## First Case Study: Auto-Publishing AI Images to Social Media

**Goal:** AI generates a featured image → publish it to both a WordPress post
and Instagram/social automatically (via n8n).

| # | Tool | Verdict | Why |
|---|---|---|---|
| 1 | Google Drive | ❌ Failed | Auth wall blocks social crawlers |
| 2 | garage-s3 | ❌ Failed | Integration issues in our setup |
| 3 | WordPress Media Library | ✅ Worked | Returns a clean, truly public direct URL |

Full write-up: [`case-studies/001-social-media-image-publishing.md`](case-studies/001-social-media-image-publishing.md)

## Contributing

Got a tool that worked — or burned you? See [CONTRIBUTING.md](CONTRIBUTING.md)
and open a PR. Every honest failure report is worth as much as a success story.

## License

MIT — use it, fork it, share your own experiences.
