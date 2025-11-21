Real-Time English-to-Japanese Subtitle Translator
Cloudflare Workers AI • Whisper ASR • Llama 3.3 Translation • Realtime • Durable Objects • Workflows

This project implements a real-time speech translation pipeline using Cloudflare’s AI and edge compute stack. Users speak into their microphone, and the system generates live Japanese subtitles with low latency. It combines audio streaming, on-edge transcription using Whisper, LLM-based translation using Llama 3.3, and Cloudflare’s Realtime Channels for bidirectional streaming of subtitles.

Overview

The system performs the following steps:

The browser captures microphone audio, encodes it as small PCM chunks, and streams it to Cloudflare via Realtime or WebSocket.

A Worker or Durable Object receives the audio chunks and runs Whisper (via Workers AI) to transcribe English speech.

The English text is passed to Llama 3.3 on Workers AI to generate a natural Japanese translation.

The translated text is streamed back to the frontend in real time.

A Durable Object maintains session state, buffers intermediate transcripts, and stores a rolling subtitle history for display or later download.

Optional Workflows coordinate multi-step translation when batching is preferred.

This architecture allows low-latency ASR and translation while maintaining a consistent session state for each user.

Features

Real-time English speech transcription using Whisper on Workers AI.

LLM-powered English-to-Japanese translation using Llama 3.3 on Workers AI.

Realtime streaming of subtitles back to the UI.

Durable Object maintaining session state and rolling subtitle history.

Frontend built on Cloudflare Pages for simple deployment.

Support for incremental streaming, partial results, and subtitle smoothing.

Uses Cloudflare Workflows for orchestrating multi-step ASR→translation pipelines.

Architecture
Frontend (Cloudflare Pages + Realtime)

Captures microphone audio using the MediaStream API.

Encodes audio frames (16kHz PCM).

Streams audio chunks to a Realtime channel or WebSocket.

Renders live translated subtitles as the backend returns results.

Workers / Durable Objects

Receive audio chunks.

Run ASR using Workers AI:

const result = await env.AI.run("@cf/openai/whisper", { audio });


Translate text using Llama 3.3:

const translated = await env.AI.run("@cf/meta/llama-3.3-70b-instruct", {
  prompt: `Translate this English text into natural Japanese:\n${result.text}`
});


Emit translation results back to the frontend via the Realtime API.

Maintain session memory, partial transcripts, and buffering.

Workflow Integration

Optional Cloudflare Workflows orchestrate:

Audio event ingestion

ASR execution

Translation

Realtime publishing

This provides auditable, fault-tolerant sequencing for each translation event.

Repository Structure
/
├── README.md
├── frontend/
│   ├── index.html
│   ├── audio.js
│   ├── realtime.js
│   └── styles.css
├── worker/
│   ├── worker.js
│   ├── durable_object.js
│   └── wrangler.toml
└── workflows/
    └── translation_workflow.json

Deployment Instructions
1. Install Wrangler
npm install -g wrangler

2. Configure wrangler.toml

Ensure the following bindings exist:

[ai]
binding = "AI"

[durable_objects]
bindings = [
  { name = "SESSION_DO", class_name = "SessionDurableObject" }
]

[vars]
REALTIME_CHANNEL = "<your-channel-name>"

3. Deploy Backend
wrangler deploy

4. Deploy Frontend
wrangler pages publish ./frontend

5. Enable Realtime

Configure a Realtime namespace in the Cloudflare dashboard and match the channel name to the environment variable.

Usage

Open the deployed Pages website.

Grant microphone access.

Start speaking English.

The interface will display real-time Japanese subtitles as they are produced.

The Durable Object manages each translation session independently, allowing simultaneous users without interference.

Notes on Latency and Optimization

Whisper-small or Whisper-tiny produce the lowest latency at the cost of slight accuracy reduction.

Subtitle smoothing uses a time-based rolling window to merge small noisy fragments into cohesive phrases.

Workflows can batch ASR events to reduce LLM overhead on longer sentences.

Realtime Channels ensure low-latency delivery, typically under 200 ms globally.
