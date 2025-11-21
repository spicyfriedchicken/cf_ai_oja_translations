# LiveSub: English → Japanese Live Subtitles on Cloudflare

LiveSub is an AI-powered application that converts live English speech into Japanese subtitles in real time using Cloudflare’s AI stack and a transformer-based translation model.

It is designed to satisfy the optional Cloudflare assignment requirements: LLM, workflow/coordination, chat or voice input, and persistent memory/state.

---

## High-Level Overview

1. User opens the LiveSub web app (Cloudflare Pages).
2. Browser captures microphone audio and streams it to a Cloudflare Worker via Realtime.
3. The Worker forwards audio chunks to a Whisper-compatible ASR endpoint (or Workers AI speech model) for English transcription.
4. Transcripts are passed to a transformer-based LLM (Llama 3.3 on Workers AI or an external model) for English → Japanese translation.
5. The Worker pushes translated subtitle lines back to the browser via Realtime events.
6. A Durable Object stores session state such as:
   - Transcript history
   - Translation history
   - User preferences
7. The web client renders synchronized subtitles.

---

## Architecture

### Frontend (Cloudflare Pages + Realtime)
- Captures microphone audio
- Streams audio chunks via Realtime
- Displays English and Japanese subtitles

### Workers / Durable Objects
- Receives audio chunks
- Runs Whisper ASR via Workers AI
- Runs Llama 3.3 translation
- Maintains session memory

### Workflow Integration
Optionally coordinates ASR → Translation → Delivery.

---

## Repository Structure

```
/
├── README.md
├── frontend/
├── worker/
└── workflows/
```

---

## Deployment

1. Install Wrangler
2. Deploy Worker with `wrangler deploy`
3. Deploy frontend with `wrangler pages publish`
4. Configure Realtime and Durable Objects

---

## License

MIT

