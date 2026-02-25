# Pipecat Voice Assistant

A real-time voice assistant built with [Pipecat](https://github.com/pipecat-ai/pipecat) and [Pipecat Flows](https://github.com/pipecat-ai/pipecat-flows). Runs in the browser via WebRTC. The bot keeps the conversation going until you explicitly say goodbye.

**Stack:** Google Gemini (LLM) · Cartesia (STT + TTS) · Silero VAD · SmallWebRTC

---

## Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/getting-started/installation/) package manager
- API keys for:
  - [Cartesia](https://cartesia.ai) — speech-to-text and text-to-speech
  - [Google AI Studio](https://aistudio.google.com/app/apikey) — Gemini LLM

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/ankitgoel2001Damco/pipecat-demo.git
cd pipecat-demo/my-pipecat-flows-app
```

### 2. Install dependencies

```bash
uv sync
```

### 3. Configure environment variables

Create a `.env` file in the `my-pipecat-flows-app` directory:

```bash
CARTESIA_API_KEY=your_cartesia_api_key
GOOGLE_API_KEY=your_google_api_key
```

---

## Running the bot

```bash
uv run main.py
```

The server starts at `http://localhost:7860`.

Open your browser and go to `http://localhost:7860` to start a conversation. The bot will greet you and keep chatting until you say "goodbye" or "stop".

---

## Project Structure

```
my-pipecat-flows-app/
├── main.py            # Bot logic and flow definitions
├── pyproject.toml     # Dependencies
├── uv.lock            # Lockfile
└── .env               # API keys (not committed)
```

## How It Works

The bot uses **Pipecat Flows** to manage conversation state:

- **Assistant node** — the bot stays in this node for the entire conversation, responding naturally to anything the user says.
- **End node** — only reached when the user explicitly says goodbye/stop/quit. The bot says a warm farewell and closes the connection.

The pipeline processes audio in real time:

```
Microphone → STT → LLM → TTS → Speaker
```
