---
name: self-healing-mcp-service
description: >-
  Build a robust host-side FastMCP (Model Context Protocol) service that survives restarts and reboots,
  reconnects cleanly to a Dockerized MCP gateway/agent, and auto-recovers its own dependency. Use when
  standing up an MCP server (tool server) for an agent — especially on macOS with launchd and a containerized
  gateway. Trigger on "build an MCP server", "MCP keeps dropping / Session not found", "MCP service won't
  stay up", "register MCP with the gateway", or "host.docker.internal MCP".
---

# Self-Healing MCP Service

A checklist + patterns for a host-side FastMCP service that doesn't fall over. Learned the hard way wiring
several MCP services into a Dockerized agent gateway.

## 1. Make the server stateless (restart resilience)
Streamable-HTTP MCP ties sessions to the **process**. If the server restarts, the gateway keeps a stale
session and every call fails with **`Session not found` (-32600)**. Fix:
```python
mcp.run(transport="http", host="0.0.0.0", port=PORT, stateless_http=True)
```
⚠️ Pass `stateless_http` to **`run()`**, NOT the `FastMCP(...)` constructor — constructor kwargs are
deprecated (2.3.4+) and crash older versions.

## 2. Supervise it (survive crashes + reboot)
Don't run it on `nohup` — it dies with the terminal and orphans. Use a process supervisor with restart:
- **macOS:** a launchd LaunchAgent with `<key>KeepAlive</key><true/>` + `<key>RunAtLoad</key><true/>`.
- **Linux:** a systemd unit with `Restart=always`.
Put the absolute interpreter path in the launcher (e.g. the conda/venv python that has your deps), and set
secrets/config via the launcher's environment block, never in code or git.

## 3. Wire it to a Dockerized gateway correctly
- The gateway (in a container) reaches a host service via **`http://host.docker.internal:PORT/mcp/`**.
- Register the **exact path that returns HTTP 200/307** — many gateways mishandle redirects; a bare `/mcp`
  may 404 while `/mcp/` works. Verify with the gateway's `mcp probe`, not raw curl (curl misses the MCP handshake).
- After restarting the host service, **re-probe / restart the gateway once** so it re-handshakes (stateless
  makes future restarts transparent).

## 4. Self-heal your own dependency
If your service depends on something that can disappear (a browser over CDP, a model server, a DB), detect it
and **relaunch it on demand** rather than failing:
```python
def _ensure_dep():
    if reachable(): return True
    subprocess.Popen([BIN, *ARGS], start_new_session=True)   # detached: survives a service restart
    # then poll until reachable, with a timeout
```
Gate every tool on this so a quit dependency recovers without human action.

## 5. Return small results
Tools feed the agent's context. **Return file paths/IDs, not big blobs** (base64 images, full documents).
If the agent must *display* a file, save it inside the agent's **workspace** (mounted + allowed by the
gateway) and return a path **relative to that workspace** — absolute host paths are usually rejected as
"outside allowed folders".

## Gotchas cheat-sheet
- `Session not found` → you forgot `stateless_http=True`, or the gateway holds a stale session (restart it once).
- Service dies overnight → it was on `nohup`; move to launchd/systemd KeepAlive.
- "Outside allowed folders" → return a workspace-relative path; save into the agent's workspace dir.
- Two instances double-process work → enforce a single instance or claim work atomically (`FOR UPDATE SKIP LOCKED`).
