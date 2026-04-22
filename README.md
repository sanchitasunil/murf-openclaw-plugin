# openclaw-murf-tts

[![npm version](https://img.shields.io/npm/v/openclaw-murf-tts.svg)](https://www.npmjs.com/package/openclaw-murf-tts)
[![ClawHub](https://img.shields.io/badge/ClawHub-openclaw--murf--tts-blue)](https://clawhub.ai/plugins/openclaw-murf-tts)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Community-managed plugin that adds Murf Falcon TTS to OpenClaw.**

OpenClaw speech-provider plugin for [Murf AI](https://murf.ai/api). Adds the `murf` speech provider to your OpenClaw workspace, calling the **FALCON** model (low-latency, ~130 ms) for high-quality, natural-sounding voice output. No local TTS stack required.

## Features

- **150+ voices** across **35 languages**
- **12 regional API endpoints** for low-latency routing
- Built-in retry with exponential backoff for `429` and `5xx` errors
- Typed errors (`MurfAuthError`, `MurfRateLimitError`, ...) for clean failure handling
- API key automatically redacted from logs

## Install

From ClawHub (recommended):

```bash
openclaw plugins install clawhub:openclaw-murf-tts
```

Or directly from npm:

```bash
npm install openclaw-murf-tts
```

`openclaw` is a peer dependency; your gateway/workspace already provides it.

## Quick start

1. Get a Murf API key from the [Murf API dashboard](https://murf.ai/api/dashboard).

2. Export the key:

   ```bash
   # macOS / Linux
   export MURF_API_KEY="your_key_here"

   # Windows PowerShell
   $env:MURF_API_KEY = "your_key_here"
   ```

3. Enable the plugin and select Murf as the TTS provider:

   ```bash
   openclaw config set plugins.entries.openclaw-murf-tts.enabled true
   openclaw config set messages.tts.provider murf
   ```

4. Restart and verify:

   ```bash
   openclaw gateway restart
   openclaw plugins list
   ```

That's it. The plugin runs with sensible defaults (`en-US-natalie` voice, FALCON model, MP3 output, 24 kHz).

## Customizing the voice

### Persist a setting with `openclaw config set`

```bash
openclaw config set messages.tts.providers.murf.voiceId en-US-jackson
openclaw config set messages.tts.providers.murf.style Newscast
openclaw config set messages.tts.providers.murf.rate 10
openclaw config set messages.tts.providers.murf.region us-east
```

Restart the gateway after changes (`openclaw gateway restart`).

### Override per message with an in-message directive

Tweak parameters inline for a single reply without touching config:

```
@tts voiceid=en-US-jackson style=Newscast rate=10 pitch=-5
```

Supported keys: `voiceid`, `model`, `style`, `rate`, `pitch`, `locale`, `format`. Each respects your `SpeechModelOverridePolicy` flags.

### Edit the JSON config directly

<details>
<summary>Example <code>openclaw.json</code></summary>

```json
{
  "messages": {
    "tts": {
      "provider": "murf",
      "providers": {
        "murf": {
          "voiceId": "en-US-jackson",
          "style": "Newscast",
          "rate": 10,
          "region": "us-east"
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

See `openclaw.config.example.json5` in the repo for a fully annotated version.

</details>

## Configuration reference

All fields live under `messages.tts.providers.murf`. All are optional; defaults are shown.

| Field        | Type   | Default         | Description |
| ------------ | ------ | --------------- | ----------- |
| `apiKey`     | string | *(env var)*     | Murf API key. Falls back to `MURF_API_KEY`. Never commit this. |
| `voiceId`    | string | `en-US-natalie` | Murf voice identifier. |
| `model`      | string | `FALCON`        | Only `FALCON` is supported in this release. |
| `locale`     | string | `en-US`         | BCP-47 locale code. |
| `style`      | string | `Conversation`  | Speaking style (e.g. `Conversation`, `Newscast`). |
| `rate`       | number | `0`             | Speech rate from `-50` to `50`. |
| `pitch`      | number | `0`             | Pitch from `-50` to `50`. |
| `region`     | string | `global`        | API region. See list below. |
| `format`     | string | `MP3`           | `MP3`, `WAV`, `OGG`, or `FLAC`. Voice-note targets always use `MP3` (FALCON returns HTTP 500 for OGG). |
| `sampleRate` | number | `24000`         | Hz. One of `8000`, `16000`, `24000`, `44100`, `48000`. |

**Regions:** `au`, `ca`, `eu-central`, `global`, `in`, `jp`, `kr`, `me`, `sa-east`, `uk`, `us-east`, `us-west`.

## Voices

Browse the full catalog in the [Murf Falcon voice library](https://murf.ai/api/products/text-to-speech/Falcon?utm_source=murf_api_docs).

Or list them from the CLI (requires the Murf provider active and a valid key):

```bash
openclaw tts voices
```

A voice ID looks like `en-US-natalie`, `en-UK-harry`, `es-ES-elvira`, etc.

## Troubleshooting

| Symptom | Fix |
| ------- | --- |
| `Murf TTS is not configured` | Set `MURF_API_KEY` in your environment, or run `openclaw config set messages.tts.providers.murf.apiKey <key>`. |
| `Murf API rejected the credentials` | Key is invalid or expired. Regenerate it in the [Murf API dashboard](https://murf.ai/api/dashboard). |
| Plugin not loading | `openclaw plugins list` should show `openclaw-murf-tts`. Check `openclaw plugins doctor` for errors. Restart the gateway after config edits. |
| Audio doesn't play in Telegram / WhatsApp | Voice-note channels expect Opus/OGG, but FALCON only emits MP3. The plugin returns playable MP3 audio, it just won't render as a native voice-note bubble. |
| Rate-limited errors | The client retries `429`/`5xx` automatically (3 attempts, exponential backoff). If it persists, check your Murf quota. |

## Development

```bash
git clone https://github.com/sanchitasunil/murf-openclaw-plugin.git
cd murf-openclaw-plugin
pnpm install
pnpm build
pnpm test
```

Live tests in `tests/live/` hit the real Murf API. They self-skip unless both `MURF_LIVE_TEST=1` and `MURF_API_KEY` are set:

```bash
MURF_LIVE_TEST=1 MURF_API_KEY=your_key pnpm test
```

## Links

- [Plugin on ClawHub](https://clawhub.ai/plugins/openclaw-murf-tts)
- [Package on npm](https://www.npmjs.com/package/openclaw-murf-tts)
- [Murf API](https://murf.ai/api)
- [Murf API dashboard](https://murf.ai/api/dashboard)
- [Murf Falcon voice library](https://murf.ai/api/products/text-to-speech/Falcon?utm_source=murf_api_docs)

## License

MIT. See [LICENSE](LICENSE).
