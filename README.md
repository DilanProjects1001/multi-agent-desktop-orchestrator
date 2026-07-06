# multi-agent-desktop-orchestrator

> A Node.js engine that orchestrates local and remote LLMs and lets them drive a real desktop — with a queue, semantic memory, and 178 tests guarding the seams.

![Stack](https://img.shields.io/badge/Node.js-18+-339933?logo=node.js&logoColor=white)
![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-integrated-6E40C9)
![Tests](https://img.shields.io/badge/tests-178%20passing-2ea44f)
![Validation](https://img.shields.io/badge/schemas-zod-3068b7)
![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)
![Status](https://img.shields.io/badge/status-portfolio%20showcase-blue)

## The problem

LLMs are great at deciding *what* to do and terrible at *reliably doing it* on a real machine. Clicks land in the wrong place, a second instance corrupts shared state, a flaky API call kills a long task, and there is no memory between runs. This project is the orchestration layer that sits between the model and the operating system and makes that loop **deterministic, resumable, and testable**.

> ℹ️ **This is a showcase repository.** The core orchestration code is the project's moat and is **not published here**. This repo ships the README, the architecture write-up, and media. The full implementation is walked through live in a technical interview.

<!--
DEMO PLACEHOLDER 1 — TEST SUITE
Capture a terminal screenshot of the full suite passing and save it as media/tests.png:
    node --test
Frame the final summary line showing "# pass 178" / "# fail 0". Then replace this comment with:
    ![178 tests passing](media/tests.png)
-->

<!--
DEMO PLACEHOLDER 2 — ORCHESTRATION GIF
Record a short GIF (8-15s) of the system in action and save it as media/orchestrate.gif:
the supervisor picking a task off the queue, the model reasoning, and the desktop being driven
(a window focusing + a click landing). Then replace this comment with:
    ![Orchestration demo](media/orchestrate.gif)
-->

## Key features

- **178 tests across 28 test files**, run with the native `node --test` runner — no external test framework. Well-covered modules include `agc_supervisor` (12 tests), `semantic_memory` (12), `agc_priority_lock` (11), `agc_prompts` (11), `command_parser` (9), `click_geometry` (8) and `claude_queue` (8).
- **Modular core — 15 modules** with single responsibilities: `agc_ledger`, `agc_supervisor`, `claude_queue`, `click_geometry`, `gui_vision`, `mcp_client`, `media_handlers`, `poll`, `process_lock`, `report_pdf`, `screen_capture`, `semantic_memory`, `text_handler`, `window_focus` and more.
- **Model Context Protocol (MCP) integration** via `@modelcontextprotocol/sdk` — the orchestrator exposes and consumes tools through MCP, so agents talk to capabilities through a standard contract instead of ad-hoc glue.
- **Semantic memory by embeddings** — tasks and outcomes are stored as vectors so the supervisor can recall relevant prior context instead of starting cold on every run.
- **Resolution-independent desktop control** — `click_geometry` maps coordinates by ratio so clicks survive a display/resolution change (a real failure mode when driving the same machine across local and remote 1080p sessions).
- **Resilience built into the task queue** — rate-limiting with `bottleneck`, automatic retries with `p-retry`, single-instance enforcement via `process_lock`, deny-by-default environment validation with `dotenv-safe`, and crash-safe persistence with `write-file-atomic`.
- **Schema-validated boundaries** — every external payload (model output, tool args, config) is parsed with `zod` before it touches application logic.

## Architecture

The system is a supervised loop: a **supervisor** pulls tasks from a **rate-limited, retrying queue**, enriches them with **semantic memory**, calls models (local via Ollama / remote via the OpenAI-compatible client) through **MCP**, and executes the resulting plan against the desktop (screen capture → GUI vision → geometry-corrected clicks → window focus / text input), appending every step to an append-only **ledger** for auditability and resume.

📐 Full breakdown — modules, data flow, and the engineering trade-offs — in **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)**.

## Tech stack

| Layer | Tools |
| --- | --- |
| Runtime | Node.js 18+, native `node --test` |
| LLM / agents | `@modelcontextprotocol/sdk` (MCP), `openai` client, local LLMs via Ollama (Llama / Qwen) |
| Memory & data | embedding-based semantic memory, `firebase-admin` |
| Desktop automation | `robotjs`, `screenshot-desktop`, `sharp`, `fluent-ffmpeg` |
| Resilience | `bottleneck` (rate limit), `p-retry` (retries), `process_lock`, `dotenv-safe`, `write-file-atomic` |
| Validation | `zod` |
| I/O & ops | `axios`, `cheerio`, `node-cron`, `node-telegram-bot-api`, `pdfkit` |

## Getting started

> The core code is private (see the showcase note above), so this repo is **read-only for evaluation**. For context, here is how the full project runs once the core is shared in an interview:

```bash
git clone <repo-url> && cd multi-agent-desktop-orchestrator
cp .env.example .env        # fill required vars — startup is deny-by-default, missing keys fail fast
npm install                 # native deps (robotjs, sharp) need build tools on the host
node --test                 # run the 178-test suite
npm start                   # boots the supervisor (single-instance lock enforced)
```

Honest note: `robotjs` and `sharp` compile native bindings, so a working toolchain (and on Windows, the desktop session it drives) is required. The suite runs headless; live desktop control does not.

## Engineering decisions / what I learned

- **Tests as the contract for a "private" repo.** Because the core isn't public, the 178 tests are how I demonstrate it actually works — they pin down geometry math, queue retry behavior, the priority lock, and prompt construction without needing the running app.
- **Treat the desktop as a hostile I/O boundary.** Resolution drift, double-execution, and partial writes are not edge cases here — they are the default. Ratio-based clicks, a single-instance lock, and atomic writes exist because each one was a real bug first.
- **Deny-by-default everywhere.** Environment validation refuses to boot on a missing secret, and `zod` rejects malformed model output at the edge, so failures surface at startup or the boundary instead of three calls deep.
- **MCP over bespoke glue.** Standardizing tool access on the Model Context Protocol made adding capabilities a contract change rather than a rewrite.

## License

Copyright (c) 2026 Dilan del Valle Mijangos. **All rights reserved.** This code is published solely for portfolio review and technical evaluation — see [LICENSE](LICENSE). No reuse, redistribution, or derivative works without prior written permission.

**Dilan del Valle Mijangos** — Full-stack developer (AI automation · hardware/NFC · security fundamentals)
📧 dilandelvallemijangos@gmail.com · 📱 951 128 8667
