# Lesson: How to Build a Good CLAUDE.md — A Three-Pass Pattern

**Applies to:** Any project where you want Claude Code to arrive already knowing
your conventions, constraints, and context — without rediscovering them every session.

---

## The Problem With Writing CLAUDE.md From Scratch

Writing a CLAUDE.md from a blank file produces generic content. You end up
describing what the repo *should* be rather than what it *actually is* — and
you miss the non-obvious things that only emerge from reading the real files.

At the same time, running `/init` alone gives you an accurate baseline but often
shallow structure: a list of commands, a brief overview, and not much else.

The pattern below fixes both problems in three passes.

---

## The Three-Pass Pattern

```
Pass 1: /init
    → grounded, repo-accurate baseline

Pass 2: Structural template
    → depth, organization, design decisions

Pass 3: Karpathy's four principles
    → agent behavior guidelines

= a CLAUDE.md that is grounded, well-structured, and behaviorally sound
```

---

### Pass 1 — `/init` (Grounding)

Run Claude Code's `/init` command. It reads your actual files and produces a
CLAUDE.md that reflects what is really in the project — real commands, real
structure, real conventions.

**What it adds:** Accuracy. The content is derived from the repo, not invented.

**What it lacks:** Depth. The output is typically a flat list of facts without
the "why" behind design decisions, without gotchas, and without behavioral guidance
for the agent itself.

Don't skip this step and write from scratch. Even if you revise everything, the
grounded starting point prevents hallucinated project details.

---

### Pass 2 — Structural Template (Depth)

Take a well-structured CLAUDE.md from another project and use it as a template
for *organization*, not content. Look for files that include:

- Per-section architecture breakdowns (not just a directory listing)
- Key design decisions with explicit rationale ("why" not just "what")
- Common gotchas — the things that will go wrong if not stated
- A data or content flow summary diagram

Rewrite your Pass 1 output into this structure, filling each section with your
repo's actual content.

**What it adds:** Organization and depth. Future Claude sessions can make judgment
calls, not just follow rules, because they understand the *reasoning* behind
conventions.

**What to avoid:** Copying the template's content. Only borrow the structure.
A CLAUDE.md for a React app should not describe Python modules.

---

### Pass 3 — Karpathy's Four Principles (Behavior)

Add an "Agent Behavior Guidelines" section built on Andrej Karpathy's four
principles, adapted to your project type. These target specific failure modes
of agentic work:

| Principle | What it prevents |
|---|---|
| **Think before acting** | Silent assumption-making; building the wrong thing confidently |
| **Simplicity first** | Over-engineering; adding features/abstractions nobody asked for |
| **Surgical changes** | Scope creep; touching adjacent code or content that wasn't part of the task |
| **Goal-driven execution** | Completing the action without solving the underlying problem |

Adapt each principle to your domain. For a code project: "no abstractions for
single-use code." For a documentation project: "change only the section that
needs updating, don't rewrite adjacent paragraphs."

**What it adds:** Behavioral guardrails. The agent now knows not just *what* to
do but *how* to behave while doing it.

Full breakdown of the principles: [`claude-md-for-ai-agent-projects.md`](claude-md-for-ai-agent-projects.md)

---

## Why Three Passes and Not One

Each pass contributes something the others can't:

| Pass | Source | Contribution |
|---|---|---|
| `/init` | Your actual repo | Accuracy — no invented details |
| Structural template | Another project's CLAUDE.md | Depth — design decisions, gotchas, flow |
| Karpathy's principles | Public research/observations | Behavior — how the agent should act |

A single-pass approach either invents content (written from scratch), stays
shallow (only `/init`), or is structurally correct but behaviorally empty
(template without principles).

---

## CLAUDE.md Is a Living Document

After the three passes, treat CLAUDE.md as something that evolves with the project:

- **Agent makes the same mistake twice** → add a gotcha
- **New convention is established** → document it
- **A section becomes stale** → update or remove it
- **A new content type or tool is added** → add it to the structure section

The goal is that every session starts with the agent already knowing what the
previous session spent time discovering.

---

## Related

- [`claude-md-for-ai-agent-projects.md`](claude-md-for-ai-agent-projects.md) — Karpathy's four principles in full
- [`CLAUDE.md`](../CLAUDE.md) — this repo's CLAUDE.md, built using this pattern
