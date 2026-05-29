# AI Companion — Production Local AI Desktop Platform

A fully local, production-ready Windows AI companion with voice interaction,
screen awareness, desktop automation, persistent memory, and adaptive learning.

**Runs entirely on your machine. No cloud. No subscriptions.**

---

## Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| OS | Windows 10 20H1 | Windows 11 |
| CPU | 4 cores, 3GHz | 8+ cores |
| RAM | 8 GB | 16–32 GB |
| GPU | None (CPU mode) | RTX 3060 12GB+ |
| VRAM | — | 8GB+ (12GB for 14B model) |
| Storage | 20 GB free | 50 GB SSD |

**VRAM guide:**
- `qwen2.5:7b`  → 5–6 GB VRAM (fits RTX 3060)
- `qwen2.5:14b` → 9–10 GB VRAM (fits RTX 3080)
- CPU fallback  → Works, ~5–10s response latency

---

## Quick Start (5 steps)

```batch
:: 1. Clone / extract the project
cd C:\Users\YourName
git clone https://github.com/yourname/ai-companion.git
cd ai-companion

:: 2. Create virtual environment
python -m venv .venv
.venv\Scripts\activate

:: 3. Run Windows setup (detects CUDA, installs deps)
python scripts/setup_windows.py

:: 4. Download models (Whisper, Piper TTS, OpenWakeWord)
python scripts/download_models.py

:: 5. Start Ollama in a separate terminal, then start the companion
:: Terminal 1:
ollama serve
:: (first run also needs: ollama pull qwen2.5:7b)

:: Terminal 2:
python main.py
```

Say **"wake up"** to activate voice input.

---

## Installation Detail

### Step 1 — Python 3.10+

Download from https://python.org — ensure "Add to PATH" is checked.

```batch
python --version   :: Should be 3.10.x or higher
```

### Step 2 — Ollama (Local LLM runtime)

Download from https://ollama.ai/download and install.

```batch
ollama serve                   :: Start the LLM server
ollama pull qwen2.5:7b         :: Download 7B model (~4.7GB)
:: Optional larger model:
ollama pull qwen2.5:14b        :: Download 14B model (~8.2GB)
```

### Step 3 — Python dependencies

```batch
python -m venv .venv
.venv\Scripts\activate
python scripts/setup_windows.py   :: Detects CUDA, installs correct packages
```

### Step 4 — GPU acceleration (optional but strongly recommended)

If you have an NVIDIA GPU with CUDA:

```batch
:: Install CUDA-enabled PyTorch (check https://pytorch.org for current command)
pip install torch==2.3.1+cu121 --index-url https://download.pytorch.org/whl/cu121

:: Replace CPU onnxruntime with GPU version
pip install onnxruntime-gpu==1.18.0
```

### Step 5 — Download AI models

```batch
python scripts/download_models.py

:: Options:
python scripts/download_models.py --whisper small        :: Better STT accuracy
python scripts/download_models.py --piper-voice en_GB-alan-medium  :: British voice
python scripts/download_models.py --no-ollama            :: Skip Ollama pull
```

---

## Usage

### Voice Commands

Say **"wake up"** then speak your command. Examples:

- *"What am I looking at on my screen?"*
- *"Help me fix this code"*
- *"Open YouTube"* → will ask for confirmation
- *"Set a reminder to review my PR in 30 minutes"*
- *"What have we talked about today?"*
- *"How productive have I been this morning?"*

### API Dashboard (text input)

```
http://127.0.0.1:8765/docs
```

Interactive API docs — send messages, clear context, check status.

### Web Dashboard

```batch
cd dashboard
npm install
npm run dev
```

Then open `http://localhost:3000`.

### CLI flags

```batch
python main.py --debug              :: Verbose logging
python main.py --no-wakeword        :: API-only mode (no mic listening)
python main.py --no-vision          :: Disable screen monitoring
python main.py --stt-model small    :: Use Whisper small (more accurate)
python main.py --llm-model qwen2.5:14b  :: Use 14B model
python main.py --personality mentor :: Change personality style
```

---

## Configuration

Edit `config/settings.toml` to change defaults.
Create `config/local.toml` to override without editing the main file.
Or set environment variables: `AI_COMPANION_LLM__MODEL=qwen2.5:14b`

Key settings:

```toml
[wake_word]
wake_words = ["wake up"]    # Change the activation phrase
threshold = 0.5             # Lower = more sensitive (more false positives)

[stt]
model = "base"              # tiny | base | small | medium | large-v3

[llm]
model = "qwen2.5:7b"        # Any Ollama model tag
temperature = 0.7

[personality]
style = "strict_coach"      # strict_coach | mentor | assistant
proactive = true            # Enable unsolicited observations
proactive_interval_sec = 300  # Minimum 5 min between proactive messages
```

---

## Startup (Windows auto-start)

### Startup registry (runs on login):
```batch
python scripts/windows_service.py startup-register
```

### NSSM service (runs as Windows service, survives logout):
```batch
:: Download nssm.exe from https://nssm.cc/download and put in PATH
python scripts/windows_service.py install
python scripts/windows_service.py start
```

### With crash recovery watchdog:
```batch
python scripts/watchdog.py
```

---

## Debugging

```batch
:: Check all dependencies
python scripts/debug_tools.py check-deps

:: List audio devices
python scripts/debug_tools.py list-audio

:: Test microphone
python scripts/debug_tools.py test-mic

:: Check GPU
python scripts/debug_tools.py check-gpu

:: Measure pipeline latency
python scripts/debug_tools.py latency

:: Run tests
pytest tests/ -v
pytest tests/ -v -k "test_permission"   :: Run specific tests
pytest tests/ -v --tb=short             :: Short tracebacks
```

### Common Issues

**"Wake word not detected"**
- Run `python scripts/debug_tools.py test-mic` to verify mic input
- Lower `threshold` in `[wake_word]` config (try 0.3)
- Check `config/settings.toml` input_device matches your mic

**"Ollama connection refused"**
- Run `ollama serve` in a separate terminal
- Check `http://localhost:11434` is accessible
- Run `ollama list` to confirm your model is downloaded

**"STT too slow"**
- Use `--stt-model tiny` for faster (less accurate) transcription
- Enable CUDA: check `python scripts/debug_tools.py check-gpu`

**"High memory usage"**
- Use `qwen2.5:7b` instead of larger models
- Set `compute_type = "int8"` in `[stt]` config

---

## Building the Installer

```batch
:: 1. Build PyInstaller executable
.venv\Scripts\activate
pyinstaller installer/companion.spec --clean

:: 2. Compile Inno Setup installer
:: (Requires Inno Setup 6 from https://jrsoftware.org)
iscc installer/setup.iss

:: Output: installer/Output/AICompanion_Setup_1.0.0.exe
```

---

## Security Model

The AI companion operates under strict security constraints:

- **All automation requires permission** — nothing executes without approval
- **Blocked forever**: shell commands, file deletion, credential access,
  registry modification, software installation, data exfiltration
- **Requires confirmation**: typing text, clicking buttons, file operations
- **Auto-approved**: opening URLs, reading clipboard, launching approved apps
- **Audit log**: every action decision is logged to `logs/audit.jsonl`
- **API bound to 127.0.0.1** — never exposed on the network without explicit config

---

## Architecture

```
Wake Word (OWW)
    ↓ WakeWordEvent
STT (Faster-Whisper)
    ↓ TranscriptionEvent
Context Manager ← Memory Engine (ChromaDB + SQLite)
    ↓             ↑ relevant memories
LLM (Ollama / Qwen2.5)  ← Vision Engine (OCR + Window detector)
    ↓ LLMStreamChunkEvent (sentence-by-sentence)
TTS (Piper)
    ↓ audio playback

[All components communicate via asyncio EventBus]
[FastAPI REST gateway on :8765 for external control]
[React dashboard on :3000]
```

---

## License

MIT License. All dependencies are open-source.
Models: Qwen2.5 (Apache 2.0), Whisper (MIT), Piper (MIT), OpenWakeWord (Apache 2.0).
