# Lesson: Write a CLAUDE.md — The Operating Manual for Your AI Agent

**Applies to:** Any project where an AI coding agent (Claude Code, Cursor, etc.)
does repeated work across multiple sessions.

**Source:** Distilled from Andrej Karpathy's public observations on agentic coding
(January 2026) and his CLAUDE.md pattern. Reference article:
https://medium.com/the-ai-studio/what-is-andrej-karpathys-claude-md-file-7ca12ef0ecec

---

## What CLAUDE.md Is

`CLAUDE.md` is a plain text file you place at the root of your project. Claude
Code reads it automatically at the start of every session — before any other
context. It is the difference between an agent that rediscovers your conventions
every session and one that arrives already knowing them.

Think of it as the operating manual you'd give a skilled contractor on day one:
not a tutorial on how to write code, but the specific rules, constraints, and
context that make *your* project different from every other project they've
worked on.

---

## Why It Matters: The Shift to Agent-Driven Development

In January 2026, Karpathy documented a shift from roughly 80% manual coding to
80% agent-driven coding in just weeks. His key observation: the bottleneck is
not the agent's raw capability — it is the absence of project-specific grounding.
An agent without a CLAUDE.md is a capable person who has to re-read the whole
codebase every time they sit down.

CLAUDE.md compresses that ramp-up into a single read.

---

## The Four Core Principles (Karpathy)

Karpathy's CLAUDE.md centers on reducing specific failure modes of agentic coding.
Each principle targets a real problem:

### 1. Think Before Coding
**Problem it solves:** Agents silently pick one interpretation of an ambiguous
request and build the wrong thing confidently.

```
Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them.
- If a simpler approach exists, say so.
- If something is unclear, stop. Name what's confusing.
```

### 2. Simplicity First
**Problem it solves:** Agents over-engineer. They add abstractions, error
handling, and flexibility that wasn't asked for, making the result harder to
maintain than what a human would have written.

```
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" that wasn't requested.
- If 200 lines could be 50, rewrite it.
```

### 3. Surgical Changes
**Problem it solves:** Agents clean up adjacent code, reformat unrelated
sections, and refactor things that weren't part of the task — creating noise
in diffs and risk of regressions.

```
- Touch only what you must.
- Don't improve adjacent code or formatting.
- Match existing style, even if you'd do it differently.
- If you notice dead code, mention it — don't delete it.
```

### 4. Goal-Driven Execution
**Problem it solves:** Tasks defined as actions ("fix the bug") can be completed
without solving the underlying problem. Reframe tasks as verifiable outcomes.

```
Transform tasks into verifiable goals:
- "Add validation" → "Write tests, then make them pass"
- "Fix the bug"    → "Reproduce it in a test, then fix"
- "Refactor X"     → "Ensure tests pass before and after"
```

---

## Applying This to Non-Code Projects

The four principles aren't code-specific. They apply equally to documentation,
knowledge base, and content repositories — anywhere an agent does repeated,
structured work.

| Principle | Code project | Documentation project |
|---|---|---|
| Think first | State assumptions before implementing | State what an entry will cover before writing it |
| Simplicity | Min code to solve the problem | Min content to document the experience; no padding |
| Surgical | Change only the function you're fixing | Change only the section that needs updating |
| Goal-driven | Tests pass before/after | Every section filled or has a TODO stub; all links resolve |

---

## What a Good CLAUDE.md Contains

Based on Karpathy's pattern and practical experience:

1. **Project overview** — what this repo is, what "work" means here
2. **Repository structure** — annotated directory tree, not just a list
3. **Content or code architecture** — the non-obvious decisions and why they exist
4. **Key design decisions with rationale** — so the agent can make judgment calls
5. **Common gotchas** — the things that will go wrong if not explicitly stated
6. **Agent behavior guidelines** — Karpathy's four principles, adapted for the project
7. **Conventions** — naming, formatting, commit messages, whatever the project needs

The last thing to include: anything that would be **non-obvious to a capable person
reading the codebase fresh**. If it can be inferred from the code or files, leave
it out.

---

## The Meta-Lesson: CLAUDE.md Is a Living Document

CLAUDE.md should evolve alongside the project. When an agent makes the same
mistake twice, add a gotcha. When a new convention is established, document it.
When a section becomes stale, update or remove it.

The goal is that every session starts with the agent already knowing what took
the previous session 20 minutes to discover.

---

## Related

- [Case Study 001 — Social Media Image Publishing](../case-studies/001-social-media-image-publishing.md)
- [Tool: n8n Community Edition](../tools/n8n-ce.md) — example of a gotcha worth documenting
- [CLAUDE.md in this repo](../CLAUDE.md) — see how we applied these principles
