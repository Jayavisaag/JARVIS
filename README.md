# 🧠 JARVIS — Autonomous Desktop AI Command Center

A fully local, voice-first AI agent for Windows — combining offline speech recognition, multimodal reasoning, computer vision, autonomous GUI control, and real-time IoT/device orchestration behind a custom-rendered neural HUD.

This isn't a chatbot wrapper. It's a closed-loop **agentic control system**: a local LLM plans actions, executes them against real system/device interfaces, observes the results, and iterates — with vision grounding, persistent memory, and safety gating on destructive operations.

![status](https://img.shields.io/badge/status-active--development-2ee6d6)
![platform](https://img.shields.io/badge/platform-Windows-0a84ff)
![python](https://img.shields.io/badge/python-3.10%2B-yellow)
![llm](https://img.shields.io/badge/inference-local--Ollama-orange)
![license](https://img.shields.io/badge/license-Private-lightgrey)

---

## ✨ Features

- 🎙️ **Hands-free wake word** — fully offline, always-listening activation, no cloud STT and no network dependency for voice input
- 🧠 **Multi-step autonomous reasoning** — plans, executes, and re-evaluates across several tool calls per request instead of answering blind in one shot
- 👁️ **Real vision grounding** — optionally sees your screen and webcam and reasons over what's actually there, not just your words
- 🖱️ **Hands-off GUI control** — can operate your desktop for you, with a spoken confirmation gate before anything risky
- 🖥️ **Full PC command surface** — voice-driven app/website launching, system volume control, live CPU/RAM/GPU/VRAM/temp monitoring, and one-click **Lock / Sleep / Restart**, plus a slide-to-confirm **Shutdown**
- 🗣️ **Natural, interruptible voice** — talk over it mid-sentence and it stops instantly, like a real conversation
- 💡 **Full smart lighting control** — power, brightness, precise RGB/HSV color, warm↔cool white balance, and scenes
- 🌀 **Custom smart fan integration** — power, 6-speed control, sleep mode, LED toggle
- 📱 **Phone as a controllable device** — launch apps, place calls, send texts, control media/volume, toggle Wi-Fi/Bluetooth/airplane mode, right from your desktop
- ⏰ **Set-and-forget scheduling** — one-off and recurring reminders/routines that fire on their own
- 🧩 **Persistent memory** — remembers facts and conversation context across sessions, not just within one chat
- 🔍 **Live web-grounded answers** — pulls current information instead of relying on stale training data
- 🖥️ **A real command-center UI** — animated neural-core visualizer, live CPU/RAM/GPU/VRAM gauges, one-click power controls, and dedicated panels for every connected device — not a bare terminal window

---

## ⚙️ Core Engineering

### Agentic Reasoning Loop
- Bounded **act → observe → act** control loop (not single-shot inference) — the model emits a tool call, the runtime executes it against a real backend, the result is folded back into context, and the model decides whether to continue or terminate.
- Structured tool/action schema parsed out of LLM output and dispatched to a typed action executor, with iteration caps to prevent runaway tool-call chains.
- Deduplication of repeated action signatures within a reasoning round to avoid redundant/looping tool invocations.
- Conditional multimodal branching: the runtime classifies whether a given turn requires visual grounding before deciding to attach image payloads, keeping token/bandwidth cost down on the common text-only path.

### Speech I/O
- Fully offline **streaming ASR** via Vosk's `KaldiRecognizer`, running continuously against a live `PyAudio` input stream for always-on wake-word detection — no network round-trip for transcription.
- Configurable multi-phrase wake-word set with continuous background listening decoupled from the main command thread.
- TTS via SAPI with a dedicated worker thread, automatic preferred-voice resolution, and a real-time **interrupt listener** — incoming mic energy during playback preempts and purges in-flight audio instead of queuing behind it.

### Vision Pipeline
- Multi-monitor screen capture (`mss`) and webcam capture (`OpenCV`) with a frame-resize + JPEG re-encode step tuned for multimodal prompt payload size.
- Coordinate-space transform layer that maps model-predicted normalized click coordinates back to absolute per-monitor pixel space, accounting for monitor offsets/resolution differences in a multi-display setup.
- Snapshot capture is decoupled onto its own thread from the main reasoning loop to avoid blocking on frame acquisition/encoding.

### Autonomous GUI Control
- Direct `pyautogui`-driven mouse/keyboard action surface exposed as first-class agent tool calls (move, click, type, hotkeys).
- A pre-execution action classifier flags actions as potentially destructive and routes them through a **spoken confirmation gate** before dispatch — the agent can propose an action but cannot silently execute high-risk ones.
- Hardware fail-safe (corner-abort) kept enabled at the automation layer independent of the agent's own gating logic — a hard interrupt path that isn't mediated by the model.

### System Control Surface
- Dedicated power-state action set (lock/sleep/restart/shutdown) routed through the same executor and confirmation gating as any other agent action, with a client-side slide-to-confirm affordance on the highest-risk operation (shutdown) as an extra deliberate-input barrier before dispatch.
- App/website launch resolved from a name-based lookup against installed applications rather than hardcoded paths, so the launcher surface adapts to the host machine.
- System volume control and live resource telemetry (CPU/RAM/GPU utilization, VRAM, temp) polled via `psutil`/GPU query and streamed to the frontend on a timer independent of the agent loop, so the HUD stays live even when the agent is idle.

### Persistence & Memory
- SQLite-backed store with a lock-guarded write path (decorator-wrapped connection handling to serialize access from multiple threads: voice loop, scheduler loop, GUI bridge).
- Rolling conversation window loaded into the prompt context on each turn, separate from a full historical log used for longer-range recall/search.
- Key/value long-term memory table the agent can read/write autonomously mid-conversation, independent of the chat transcript.
- Custom trigger-phrase → action mappings persisted and matched before falling back to full LLM reasoning, giving near-zero-latency handling for frequently used commands.

### Scheduler
- Background scheduler thread polling for due tasks independent of the main agent loop, supporting both one-shot fire times and recurring (day-of-week + hour/minute) schedules.
- Task execution routed through the same action-execution path as live agent actions, so scheduled tasks and conversational commands share one code path and one safety layer.

### Device Control Layer
- Async smart-lighting control (`pywizlight`) run on a dedicated asyncio event loop bridged into the primarily synchronous/thread-based agent runtime, with full HSV/color-temp/brightness/scene control.
- Custom UDP-datagram protocol implementation for a non-standard smart fan (power/6-step speed/sleep/LED), independent of any vendor SDK.
- ADB-based Android control surface — app lifecycle, telephony, media/volume transport, radio toggles (Wi-Fi/BT/airplane/mobile data/hotspot), and input injection — with connection-state detection and retry/backoff handling.

### Frontend / HUD
- Frameless custom-chrome window (drag regions, resize edges, native-style controls) rendered entirely in HTML/CSS/JS and bridged to the Python backend via a thin RPC surface.
- Canvas + SVG composited "neural core" visualizer (mesh shading, ring geometry, orbital nodes) driven by live status/state rather than being purely decorative.
- Real-time system telemetry (CPU/RAM/GPU/VRAM/temp) polled and pushed into animated SVG gauges.
- Bidirectional state sync for device panels (lighting color space, fan speed, phone status) so the UI reflects out-of-band state changes, not just its own commands.

---

## 🛠️ Stack

| Layer | Technology |
|---|---|
| Desktop shell / IPC | `pywebview` |
| Runtime | Python 3.10+, multi-threaded (voice / vision / scheduler / GUI bridge) |
| LLM inference | Local, via `Ollama`, multimodal-capable model |
| ASR | `Vosk` (offline, streaming) + `PyAudio` |
| TTS | Windows SAPI |
| Vision capture | `OpenCV`, `mss`, `PIL` |
| GUI automation | `PyAutoGUI` |
| Async device I/O | `asyncio` + `pywizlight` |
| Fan control | Custom UDP protocol |
| Phone control | Android Debug Bridge |
| Web search grounding | `ddgs` (keyless) |
| Persistence | SQLite (thread-guarded) |
| Frontend | HTML5, CSS3, vanilla JS, Canvas/SVG |

---

## 🚧 Status

Actively evolving personal build — Windows-first, with hardware-specific assumptions baked in (particular smart-light/fan protocols, ADB-based phone bridge). Shared as a portfolio/reference project rather than a turnkey install.

---

## ⚠️ Disclaimer

This system executes real actions against a live machine and network — process control, input injection, and networked devices. It's designed for personal use on a trusted local setup; review and understand the code before running it on your own hardware. 
 
