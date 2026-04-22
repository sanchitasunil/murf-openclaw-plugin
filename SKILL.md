---
name: Murf Falcon TTS
slug: openclaw-murf-tts
description: High-quality text-to-speech for OpenClaw via Murf AI Falcon (low-latency, ~130 ms).
required_env:
  - MURF_API_KEY
---

# Murf Falcon TTS

High-quality text-to-speech for OpenClaw via [Murf AI](https://murf.ai/api).

> **Required:** `MURF_API_KEY`. Get one from the [Murf API dashboard](https://murf.ai/api/dashboard).

## What it does

Adds the **murf** speech provider to your OpenClaw workspace. Outbound messages (or inbound, depending on your `auto` setting) are synthesized into natural-sounding audio using Murf's Falcon model.

- **FALCON** model: low latency (~130 ms), conversational quality, 24 kHz default
- **150+ voices** across 35 languages
- **12 regional endpoints** for low-latency routing

## Install

From ClawHub (recommended):

```bash
openclaw plugins install clawhub:openclaw-murf-tts
```

Or directly from npm:

```bash
npm install openclaw-murf-tts
```

## Setup

1. Get a Murf API key from the [Murf API dashboard](https://murf.ai/api/dashboard).
2. Set `MURF_API_KEY` in your environment.
3. Enable the plugin and select Murf as the TTS provider:

   ```bash
   openclaw config set plugins.entries.openclaw-murf-tts.enabled true
   openclaw config set messages.tts.provider murf
   ```

4. Or add directly to your OpenClaw config:

   ```json
   {
     "messages": {
       "tts": {
         "provider": "murf",
         "providers": {
           "murf": {
             "voiceId": "en-US-natalie"
           }
         }
       }
     },
     "plugins": {
       "enabled": true,
       "entries": {
         "openclaw-murf-tts": { "enabled": true }
       }
     }
   }
   ```

5. Restart the gateway: `openclaw gateway restart`
6. Verify: `openclaw plugins list` should show `openclaw-murf-tts`

## Configuration

All fields live under `messages.tts.providers.murf`.

| Field | Default | Description |
|-------|---------|-------------|
| `apiKey` | `MURF_API_KEY` env | Murf API key. Falls back to the env var. Never commit this. |
| `voiceId` | `en-US-natalie` | Voice identifier. |
| `model` | `FALCON` | Only `FALCON` is supported. |
| `locale` | `en-US` | BCP-47 locale. |
| `style` | `Conversation` | Speaking style (e.g. `Conversation`, `Newscast`). |
| `rate` | `0` | Speech rate from -50 to 50. |
| `pitch` | `0` | Pitch from -50 to 50. |
| `region` | `global` | API region. See list below. |
| `format` | `MP3` | `MP3`, `WAV`, `OGG`, or `FLAC`. Voice-note targets always use MP3. |
| `sampleRate` | `24000` | Sample rate in Hz. One of 8000, 16000, 24000, 44100, 48000. |

## In-message directives

```
@tts voiceid=en-US-jackson style=Newscast rate=10 pitch=-5
```

Supported keys: `voiceid`, `model`, `style`, `rate`, `pitch`, `locale`, `format`.

## Regions

`au`, `ca`, `eu-central`, `global`, `in`, `jp`, `kr`, `me`, `sa-east`, `uk`, `us-east`, `us-west`

## Voices

Browse the full catalog in the [Murf Falcon voice library](https://murf.ai/api/products/text-to-speech/Falcon?utm_source=murf_api_docs).

Or list them from the CLI:

```bash
openclaw tts voices
```

## Links

- [Plugin on ClawHub](https://clawhub.ai/plugins/openclaw-murf-tts)
- [Package on npm](https://www.npmjs.com/package/openclaw-murf-tts)
- [Murf API](https://murf.ai/api)
- [Murf API dashboard](https://murf.ai/api/dashboard)
- [Murf Falcon voice library](https://murf.ai/api/products/text-to-speech/Falcon?utm_source=murf_api_docs)
- [GitHub](https://github.com/sanchitasunil/murf-openclaw-plugin)
