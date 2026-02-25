# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

The repo contains a single application under `my-pipecat-flows-app/`:

- `main.py` — entry point and all application logic
- `pyproject.toml` — project metadata and dependencies (managed with `uv`)
- `.python-version` — pins Python 3.12+

## Commands

All commands should be run from `my-pipecat-flows-app/`.

```bash
# Run the bot
uv run main.py

# Add a dependency
uv add <package>
```

## Required Environment Variables

Create a `.env` file in `my-pipecat-flows-app/`:

```
CARTESIA_API_KEY=...
GOOGLE_API_KEY=...
```

## Architecture

This is a **Pipecat Flows** voice bot demo. The key dependency is `pipecat-ai-flows`.

### Pipeline

Audio frames flow through a linear pipeline:

```
transport.input() → STT (Cartesia) → context_aggregator.user() → LLM (Google) → TTS (Cartesia) → transport.output() → context_aggregator.assistant()
```

`LocalSmartTurnAnalyzerV3` with `TurnAnalyzerUserTurnStopStrategy` handles end-of-turn detection instead of simple silence-based VAD cutoff.

### Flows / State Machine

Conversation logic is expressed as a graph of `NodeConfig` objects managed by `FlowManager`:

- Each **node** has `role_messages` (persistent system prompt), `task_messages` (per-turn instructions), `functions` (tools the LLM can call), and optional `post_actions` (e.g., `end_conversation`).
- **Functions** are `FlowsFunctionSchema` objects whose `handler` is an `async def` returning `(result, next_node)` — the return value is sent back to the LLM and the returned node becomes the active node.
- `FlowManager` is wired into the `PipelineTask`, `LLMService`, `LLMContextAggregatorPair`, and `BaseTransport`.
- Conversation starts by calling `await flow_manager.initialize(create_initial_node())` inside the `on_client_connected` transport event handler.

### Transport Abstraction

`create_transport(runner_args, transport_params)` from `pipecat.runner.utils` selects between Daily (WebRTC room), Twilio (FastAPI WebSocket), and generic WebRTC transports based on CLI arguments. Each transport uses `SileroVADAnalyzer` for voice activity detection.

The `bot(runner_args)` async function is the Pipecat Cloud-compatible entry point; running `main.py` directly invokes `pipecat.runner.run.main()` for local execution.
