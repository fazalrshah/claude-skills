---
name: living-docs-workflow
description: >-
  Never lose context across long, multi-session, or multi-chat AI-assisted projects. Use when a build
  spans many chats/days and decisions, bugs, and state keep getting re-explained or lost. Sets up a set
  of hardcoded markdown "source of truth" files + pickup rules + session snapshots so any new chat resumes
  fully current. Trigger when the user says "I keep re-explaining", "we lost context", "resume", "save the
  session", or is starting a long build.
---

# Living-Docs Workflow

A discipline for AI-assisted projects that outlive a single context window: **nothing lives in the model's
memory — everything lives in versioned markdown.** Any fresh chat reads a fixed set of files and is instantly
caught up.

## The files (create these at the project root)
- **`ARCHITECTURE.md`** — the single source of truth: the system map, locked decisions, stack, and gotchas.
  If a decision changes, update THIS file (not just chat) so future chats inherit it.
- **`CLAUDE.md`** — the **pickup rules**: an ordered "read these first in every new chat" list pointing at the
  files below, plus auto-save rules. This is what makes resume reliable.
- **`session-logs/session.md`** — append-only running log: a timestamped entry after every meaningful action
  (code written, command run, bug fixed, decision made). Never overwrite.
- **`issues.md`** — observed bugs as `ISS-NNN`, each **OPEN until re-verified** (🔴/🟡/🟢 with date + evidence).
- **`notes.md`** — improvement backlog / ideas, tagged to the relevant component, surfaced on demand.
- **`tests.md`** — acceptance/verification checklist to run when an area is "done".
- (optional) `Contracts.md`, `security.md`, `rag.md`, `opensource-ideas.md` for domain-specific tracks.

## Pickup rules (put in CLAUDE.md)
1. Read `ARCHITECTURE.md` (source of truth). 2. Read the tail of `session-logs/session.md`. 3. Read `CLAUDE.md`.
4. Read `notes.md` (surface relevant items per component). 5. Read `issues.md` (re-verify open issues in-area).
Add a one-line **PENDING PRIORITY banner** at the top of CLAUDE.md when a specific next task is agreed, so the
next chat surfaces it even if the user opens a different topic.

## Session snapshots (resume)
On "save session", write a dated snapshot to a snapshots dir capturing: what worked (with evidence), what
failed/gotchas, exact next step, and environment notes. On "resume", load the latest snapshot.

## Rules that make it work
- **Update the doc, not just chat.** A decision only spoken in chat is lost next session.
- **Append, never overwrite** the session log.
- **Issues stay OPEN until re-tested** — flip to 🟢 only with date + evidence.
- **Capture, don't carry.** The moment you notice a reusable lesson or a bug, write it to the right file.

## Why it works
Multi-chat AI builds fail when context is lost between windows. A fixed "read these N files first" contract +
append-only logs + dated snapshots means every instance is fully current in seconds, decisions never drift,
and bugs don't silently reappear.
