# Changelog

All notable changes to this fork of [`googlecolab/colab-mcp`](https://github.com/googlecolab/colab-mcp) are documented here.

This fork follows the upstream `1.0.x` baseline and tags fork-specific work
with the date of the change. Upstream-merged work keeps its own commit
history.

## 2026-05-14 — Stale-server detection and cleanup

Adds a process registry that solves the "Disconnected from the local Colab MCP server" issue ([upstream discussion #84](https://github.com/googlecolab/colab-mcp/discussions/84)) caused by orphaned servers from prior Claude Code sessions.

### Added
- `src/colab_mcp/process_registry.py` — cross-platform process registry (stdlib only).
  Tracks `{pid, port, host, started_at}` for each running server in:
  - `%LOCALAPPDATA%\colab-mcp\registry.json` on Windows
  - `~/.colab-mcp/registry.json` on macOS/Linux
- CLI flag `--list-running` — print every currently-running `colab-mcp` server.
- CLI flag `--kill-stale` — terminate orphaned `colab-mcp` servers and exit.
- Automatic registration on server startup (after the WebSocket port is bound)
  and unregistration on clean shutdown.
- Automatic pruning of dead registry entries on every startup.

### Changed
- `open_colab_browser_connection` timeout error now reports this server's port
  *and* the PIDs/ports of any peer `colab-mcp` processes, so users can
  immediately recognize the "old browser tab pointed at a dead port" case
  instead of staring at a generic "timed out" message.

### Docs
- README: new **CLI Reference** section, new troubleshooting entry, new row in
  "What's Different" table.
- Changes from Upstream: stale-server detection bullet added.

### Verification
Verified end-to-end on Windows 10 with `uv run` and `uvx`:
- Module loads under uv-managed venv (no import errors).
- `--list-running` correctly reports no servers when none registered.
- `--kill-stale` removes a synthetic registry entry whose PID is dead
  (`Terminated 1 stale colab-mcp server(s): pid=888888 port=55555`) and
  empties the registry file.
- Real `colab-mcp` server startup logs `Registered colab-mcp pid=X port=Y`.
- Clean shutdown removes the server's own entry (subsequent `--list-running`
  shows none).

## Earlier (upstream + previous fork commits)

See `git log` for the full history. Highlights:

- `f70c00d` — pre-register all 5 notebook tools at startup (fixes invisible
  tools in MCP clients that don't support `notifications/tools/list_changed`).
- `cae498b` — add `change_runtime` tool with OAuth for programmatic GPU
  assignment.
- `e66ee69` — match real Colab API signatures (language param, cellId,
  run_code_cell).
- `440e3bc` — fix `ColabClient` initialization (missing `Prod()` env arg) +
  change OAuth port to 8085 for Windows.
