# claude-skills

A small collection of **Claude Code skills** — battle-tested patterns extracted from building a real,
local-first, multi-agent system. Each skill is a self-contained `SKILL.md` (with triggers) that Claude Code
loads on demand to do a task or apply hard-won guidance. Clean-room and dependency-light.

> A "skill" is just a folder with a `SKILL.md` (name + description + instructions, optionally scripts).
> Claude Code auto-loads it when the description matches what you're doing.

## Install

```bash
git clone https://github.com/fazalrshah/claude-skills.git
cp -r claude-skills/* ~/.claude/skills/      # or copy individual skill folders you want
```

Then in any Claude Code session, the skill activates automatically when its description matches your task
(e.g. you ask to "build an MCP server" → `self-healing-mcp-service` kicks in).

## Skills

| Skill | Type | What it gives you |
|-------|------|-------------------|
| **[living-docs-workflow](living-docs-workflow/SKILL.md)** | methodology | Never lose context across long, multi-chat AI builds — a "nothing in memory, everything in markdown" system (pickup rules + issue/notes/tests trackers + resumable session snapshots). |
| **[self-healing-mcp-service](self-healing-mcp-service/SKILL.md)** | setup | Build an MCP (tool) server that survives restarts/reboots, reconnects cleanly to a Dockerized gateway, and auto-heals its own dependency. Includes the `stateless_http`, launchd KeepAlive, `host.docker.internal`, and "Session not found" / "outside allowed folders" gotchas. |
| **[coverage-verified-rag-indexer](coverage-verified-rag-indexer/SKILL.md)** | setup | Build a local RAG indexer that **never silently drops chunks** — block-structured stages with per-stage error attribution, a coverage check (`stored == chunks`) before any job completes, OCR-capable extraction, token-based chunking, and an atomic job claim. |

## Roadmap

Planned additions (contributions welcome):
- `local-llm-apple-silicon` — evidence-based local-model selection on Apple Silicon (MoE vs dense, MLX, RAM/tool-calling tradeoffs).
- `regulated-ai-agent` — the locked-down-agent + deterministic-gate pattern for compliance-sensitive autonomous agents.
- `agent-contract-template` — generate a per-agent SLA/Contract block (operating bounds, fail-safe, escalation, output ownership) for autonomous agents.
- `append-only-audit-ledger` — a tamper-evident Postgres audit-trail pattern.
- `mcp-launchd-scaffold` — one command to scaffold a production-ready FastMCP service.
- `brand-consistent-image-gen` — image-generation MCP with a brand prompt template + reference images + a "good prompt" library.

## Related projects
- [**airlock**](https://github.com/fazalrshah/airlock) — deterministic, self-healing egress gate (MCP) for locked-down LLM agents.
- [**mosaic**](https://github.com/fazalrshah/mosaic) — coverage-verified local RAG indexer (the reference implementation behind `coverage-verified-rag-indexer`).

## License
MIT — see [LICENSE](LICENSE).

<!-- fazalshah-personal-site:start -->
## Author and engineering notes

Published by [Fazal Shah](https://fazalshah.com/). Explore my local AI experiments, Mac hardware test notes, agent engineering articles, and public tools on my [personal website](https://fazalshah.com/).

[All public repositories](https://github.com/fazalrshah?tab=repositories) · [YouTube builds and walkthroughs](https://www.youtube.com/@fazalrshah)
<!-- fazalshah-personal-site:end -->
