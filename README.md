# 🧠 JARVIS — Personal AI Command Center

A fully local, voice-first desktop AI assistant inspired by *Iron Man's JARVIS* — built as a frameless native-feeling desktop app with a live HUD, wake-word voice control, computer vision, smart home integration, and a self-driving agent loop that can *see* your screen and *act* on it.

Unlike typical voice assistants, JARVIS isn't just a chatbot with a mic — it's an **agentic control layer for your PC and home**, combining offline speech recognition, a local LLM brain, screen/webcam vision, GUI automation, and real-time device control behind a single glowing neural-core interface.

![status](https://img.shields.io/badge/status-active--development-2ee6d6)
![platform](https://img.shields.io/badge/platform-Windows-0a84ff)
![python](https://img.shields.io/badge/python-3.10%2B-yellow)
![license](https://img.shields.io/badge/license-Private-lightgrey)

---

## ✨ Highlights

- 🎙️ **Always-listening wake word** (`"Hey JARVIS"`, `"Friday"`, `"Computer"`, ...) using fully offline Vosk speech recognition — no cloud STT required
- 🧠 **Agentic reasoning loop** — an act → observe → act cycle that lets JARVIS chain multiple tool calls per request instead of one-shot replies
- 👁️ **Live vision system** — optional webcam + screen capture piped straight into a multimodal LLM so JARVIS can actually *see* what you're looking at
- 🖱️ **Autonomous GUI control** — JARVIS can move the mouse, click, type, and operate your desktop, with spoken confirmation gating for anything destructive
- 🗣️ **Natural TTS voice** with interrupt-to-speak support (talk over JARVIS and it shuts up instantly)
- 💡 **Smart home control** — WiZ smart lighting (full RGB/HSV/color-temp/scene control) and a custom UDP-based smart fan
- 📱 **Android phone bridge** over ADB — launch apps, place calls, control media/volume, toggle Wi-Fi/Bluetooth/airplane mode, send texts, and more
- ⏰ **Scheduler & reminders** — one-off and recurring task scheduling with a persistent SQLite-backed queue
- 🧩 **Long-term memory** — SQLite-backed conversation history and key/value memory so JARVIS remembers context across sessions
- 🔍 **Live web search** — keyless, free web search grounding for up-to-date answers
- 🖥️ **Custom-built desktop GUI** — frameless glassmorphic command center with animated neural-core visualizer, real-time CPU/RAM/GPU telemetry gauges, and full system power controls, all rendered in HTML/CSS/JS and bridged to Python via `pywebview`

---

## 🏗️ Architecture

```
┌─────────────────────────────┐        ┌───────────────────────────┐
│        Frontend (GUI)        │        │        Backend (Agent)      │
│  index.html / app.js / css   │◄──────►│      Jarvis_agent.py         │
│  Neural visualizer, gauges,  │  JS-API │  Wake word · STT · TTS       │
│  device panels, transcript   │  bridge │  LLM reasoning loop · Vision │
└──────────────┬───────────────┘        │  GUI automation · Scheduler  │
               │                        │  Smart home / phone control  │
        pywebview window                └───────────────┬───────────┘
               │                                          │
               └──────────────── gui_bridge.py ───────────┘
                     (exposes a thin `Api` class as
                      window.pywebview.api to the JS frontend)
```

- **`Jarvis_agent.py`** — the core agent: wake-word detection, offline STT (Vosk), TTS (SAPI), the local LLM reasoning/tool-use loop, vision capture and encoding, GUI automation, smart light/fan/phone control, scheduling, and a SQLite persistence layer for memory and conversation history.
- **`gui_bridge.py`** — a `pywebview`-based desktop shell. Exposes a clean `Api` class (window controls, system stats, device control, transcript streaming, scheduled tasks, etc.) as a JS bridge, and boots the whole app as a native window.
- **`index.html` / `app.js` / `style.css`** — the command center UI: a custom frameless titlebar, animated SVG/canvas "neural core" globe visualizer, live telemetry gauges, transcript/chat panel, and dedicated control panels for lighting, fan, phone, and app launching.

---

## 🧩 Core Systems

### 🎙️ Voice Pipeline
- Continuous offline wake-word listening via **Vosk + PyAudio**
- Configurable wake phrases
- Barge-in support — speaking while JARVIS talks interrupts playback instantly
- Natural voice synthesis with automatic preferred-voice detection

### 🤖 Reasoning Engine
- Runs against a **local LLM via Ollama**, keeping everything on-device
- Multi-round **act → observe → act** agent loop (not just single-shot Q&A) so JARVIS can plan, execute an action, observe the result, and decide the next step
- Vision-aware prompting: automatically decides when a request needs an image and attaches webcam/screen context
- Pluggable **custom voice commands** stored in SQLite (map a trigger phrase straight to an action)

### 👁️ Vision
- Webcam capture + multi-monitor screen capture via `mss`
- Frame resizing/compression pipeline for efficient multimodal prompts
- Coordinate scaling so the AI's "click here" coordinates map correctly across monitors

### 🖱️ GUI Automation
- `pyautogui`-driven mouse/keyboard control exposed as agent actions
- Built-in **danger detection** — potentially destructive actions require a spoken confirmation before executing
- PyAutoGUI fail-safe enabled (mouse-to-corner emergency stop)

### 💡 Smart Home
- **WiZ lights**: power, brightness, full RGB/HSV color, warm↔cool white balance, and scene modes — controlled asynchronously via `pywizlight`
- **Custom smart fan**: UDP protocol control for power, 6-speed control, sleep mode, and LED toggle

### 📱 Phone Control (ADB)
- Full Android bridge: launch/kill apps, place calls to saved contacts, send text, media transport controls, volume, Wi-Fi/Bluetooth/airplane mode/hotspot/mobile data toggles, Google/YouTube search, and navigation (home/back/recents)

### ⏰ Scheduler & Memory
- Persistent SQLite database for:
  - Conversation history (rolling context + full searchable log)
  - Key/value long-term memory
  - Custom command mappings
  - One-off and recurring scheduled tasks (with day-of-week support)
- Background scheduler loop that fires due tasks automatically

### 🖥️ Command Center UI
- Frameless, custom-drawn window with drag region, resize handles, and native-style min/max/close controls
- Animated **neural core visualizer** (canvas mesh + SVG rings + orbiting nodes) that reflects live status
- Real-time **CPU / RAM / GPU temp** gauges plus dedicated GPU utilization/VRAM panel
- One-click **Lock / Sleep / Restart**, and a slide-to-confirm **Shutdown** control
- Full **smart light control panel** with a color picker (SV box + hue bar), hex/RGB inputs, and warm/cool slider
- **Fan**, **phone**, and **app launcher** panels
- Live transcript panel with a toggle for whether webcam/screen images are sent to the AI

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Desktop shell | `pywebview` |
| Backend | Python 3.10+ |
| LLM | Local model via `Ollama` (multimodal-capable) |
| Speech-to-text | `Vosk` + `PyAudio` (offline) |
| Text-to-speech | Windows SAPI |
| Vision | `OpenCV`, `mss`, `PIL` |
| GUI automation | `PyAutoGUI` |
| Smart lighting | `pywizlight` (WiZ protocol) |
| Smart fan | Custom UDP protocol |
| Phone control | Android Debug Bridge (ADB) |
| Web search | `ddgs` (free, keyless) |
| Persistence | SQLite |
| Frontend | HTML5, CSS3, vanilla JS, SVG/Canvas |

---

## 🚧 Status

This is an actively evolving personal project — expect rapid iteration, rough edges, and hardware-specific assumptions (Windows-first, WiZ lighting, custom fan protocol, Android via ADB). It's shared here as a portfolio/reference project rather than a plug-and-play install.

---

## ⚠️ Disclaimer

This project automates real system actions (shutdown/restart, mouse/keyboard control, smart devices, phone control). It's built for personal use on a trusted local machine — review the code before running it on your own setup.
