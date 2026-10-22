## Formatting

After cloning: `uv sync` then `uv run pre-commit install`. The hook runs
mdformat on the Markdown and ruff on `salva` and `converti`, which have no `.py`
extension and are named in `.pre-commit-config.yaml`.

## Configured with dhevmera

This repository's Claude Code configuration — this file, `.claude/memory/`,
`.claude/settings*.json` — is managed with dhevmera. Read `~/.claude/CLAUDE.md`,
the global instructions, first; nothing here repeats it.

Last reviewed on 2026-10-22 against dhevmera 34a1b8bd0022.
