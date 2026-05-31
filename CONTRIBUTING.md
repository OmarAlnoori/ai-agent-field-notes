# Contributing

Thanks for wanting to add to this library. The whole point is real experience,
so a short honest write-up of something that burned you is more valuable than a
polished tutorial.

## What We're Looking For

- **Case studies** — a goal, the tools you tried (in order), what happened, and
  what you ended up shipping. Failures are especially welcome.
- **Tool pages** — a focused verdict on a single tool: what it's good for, what
  breaks it, and a working (or known-broken) config snippet.
- **Lessons** — a cross-cutting principle you extracted after hitting the same
  problem across multiple tools or workflows.

## Before You Open a PR

1. Check that a similar entry doesn't already exist. If one does, consider
   editing it to add your experience rather than duplicating it.
2. Use the template for your content type (see below).
3. Keep it grounded — link to error messages, screenshots, or logs where you can.
   Vague "it didn't work" entries are hard for others to act on.
4. Don't redact the failure. The exact error, the misunderstanding, the wasted
   hours — that's the signal.

## Case Studies

Copy [`case-studies/TEMPLATE.md`](case-studies/TEMPLATE.md) to a new file named
`case-studies/NNN-short-slug.md` (increment the number). Fill in every section;
leave a `> TODO` note if you genuinely don't know something rather than omitting
the section entirely.

## Tool Pages

Create or edit a file in `tools/<tool-name>.md`. Structure it as:

```
# Tool Name

**Verdict:** ✅ Worked / ⚠️ Partial / ❌ Failed (in what context)

## What It Is
## Why We Tried It
## What Happened
## Config / Code Snippet (if applicable)
## Verdict Detail
## See Also
```

## Lessons

Create or edit a file in `lessons/<short-slug>.md`. Lessons should name a
concrete principle, not just re-tell a story. Cross-link the case studies that
informed it.

## Style Guide

- Use plain Markdown. No fancy formatting required.
- Write in first-person plural ("we tried", "we found") or neutral present
  ("Drive blocks social crawlers"). Either is fine; mixing is fine too.
- Verdicts use one of three icons: ✅ Worked · ⚠️ Partial · ❌ Failed.
- Tool names match the file names in `tools/` so cross-links stay consistent.

## Submitting

Open a pull request against `main`. The title should be:

```
add: <short description>        # new entry
update: <tool or case study>    # editing existing content
fix: <what you fixed>           # typo / broken link / factual correction
```

No CLA, no contributor agreement. MIT license applies to all contributions.
