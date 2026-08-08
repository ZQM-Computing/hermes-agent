# hermes-agent

[![CI](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml/badge.svg)](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml) [![Tests](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/tests.yml/badge.svg)](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/tests.yml) [![Ruff](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml/badge.svg)](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml) [![mypy](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml/badge.svg)](https://github.com/ZQM-Computing/hermes-agent/actions/workflows/ci.yml)


The self-improving AI agent built by Nous Research. It creates skills from experience, improves them during use, and runs anywhere.

## About

`hermes-agent` is a personal AI agent that runs the same agent core across a CLI, a messaging gateway (Telegram, Discord, Slack, and ~20 other platforms), a TUI, and an Electron desktop app. It learns across sessions via memory and skills, delegates to subagents, runs scheduled jobs, and drives a real terminal and browser. Capability is extended primarily through plugins and skills rather than core growth.

Python 3.11+ is required; the exact-pinned dependency set and lockfile are load-bearing for supply-chain safety.

## Installation

```bash
# installer script (Linux, macOS, WSL2)
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# Windows PowerShell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)

# manual / editable install
uv venv ~/.hermes/venvs/hermes-dev --python 3.11
uv pip install -e ".[all,dev]"
```

## Usage

```bash
hermes              # interactive CLI
hermes model        # choose provider/model
hermes tools        # configure tools
hermes config set   # set config values
hermes gateway      # start messaging gateway
hermes setup        # full setup wizard
hermes update       # update
hermes doctor       # diagnostics
```

## Features

- CLI, TUI, gateway, and desktop app from a single codebase
- Built-in learning loop: memory, skills, session search
- Scheduled cron jobs with cross-platform delivery
- Subagent delegation and parallel workstreams
- Browser CDP supervisor and real terminal backend
- MCP integration and plugin system
- Supply-chain hardened exact-pinned installs
- Native Windows support without WSL

## CI Badges

- Docs: https://img.shields.io/badge/Docs-hermes--agent.nousresearch.com-FFD700
- Discord: https://img.shields.io/badge/Discord-5865F2
- License: https://img.shields.io/badge/License-MIT-green
- Built by Nous Research: https://img.shields.io/badge/Built%20by-Nous%20Research-blueviolet

## Integration: zqm-intel-platforms

`hermes-agent` integrates with `zqm-intel-platforms` via messaging, tool, and memory surfaces.

## License

MIT — see [LICENSE](LICENSE).

## Contact

zqmcomputing@gmail.com

## Related Repositories

- [ZQM-Computing/hermes](https://github.com/ZQM-Computing/hermes) — CLI runtime and session management for Hermes
- [ZQM-Computing/hermes-config](https://github.com/ZQM-Computing/hermes-config) — profiles, skills, and MCP server configs
- [ZQM-Computing/swarm](https://github.com/ZQM-Computing/swarm) — multi-agent mesh orchestration for distributed Hermes workloads
- [ZQM-Computing/zqm-ai-master](https://github.com/ZQM-Computing/zqm-ai-master) — FastAPI gateway with Ollama inference and AI council
- [ZQM-Labs/ollama-bridge](https://github.com/ZQM-Labs/ollama-bridge) — MCP bridge for capability-aware model routing to Ollama hosts
- [ZQM-Labs/zqm-hermes-skills](https://github.com/ZQM-Labs/zqm-hermes-skills) — Hermes skills for networking, Windows, LAN, GitHub, and automation
