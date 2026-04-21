# openclaw-murf-tts

> **Community-managed plugin to add Murf Falcon TTS to OpenClaw.**

OpenClaw speech-provider plugin for [Murf AI](https://murf.ai) Falcon TTS.

Adds the `murf` speech provider to your OpenClaw workspace, calling Murf's
streaming `/v1/speech/stream` endpoint with the **FALCON** model
(low-latency, ~130 ms) so messages get high-quality, natural-sounding voice
output without you running your own TTS stack.

- 150+ voices across 35 languages
- 12 regional API endpoints
- Built-in retry/backoff for `429`/`5xx`
- Typed errors (`MurfAuthError`, `MurfRateLimitError`, ...) for clean
  failure handling
- API key never logged

## Install

```bash
openclaw plugins install openclaw-murf-tts
```

Or directly from npm:

```bash
npm install openclaw-murf-tts
```

`openclaw` is a peer dependency -- your gateway/workspace already provides it.

## Quick start

You can set everything up from the command line. No config-file editing
required.

1. Get a Murf API key from the [Murf API dashboard](https://murf.ai/api/dashboard).

2. Export the key in your shell:

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

4. Restart the gateway:

   ```bash
   openclaw gateway restart
   ```

5. Verify it loaded:

   ```bash
   openclaw plugins list
   openclaw plugins doctor
   ```

That's it -- the plugin runs with sensible defaults
(`en-US-natalie` voice, FALCON model, MP3 output, 24 kHz).

## Customizing the voice

Three options, from least invasive to most:

### 1. Persist a setting with `openclaw config set`

```bash
openclaw config set messages.tts.providers.murf.voiceId en-US-jackson
openclaw config set messages.tts.providers.murf.style Newscast
openclaw config set messages.tts.providers.murf.rate 10
openclaw config set messages.tts.providers.murf.region us-east
```

Restart the gateway after changes (`openclaw gateway restart`).

### 2. Override per message with an in-message directive

When OpenClaw allows directive overrides, you can tweak parameters inline
for a single reply without touching config:

```
@tts voiceid=en-US-jackson style=Newscast rate=10 pitch=-5
```

Supported directive keys: `voiceid`, `model`, `style`, `rate`, `pitch`,
`locale`, `format`. Each one respects your `SpeechModelOverridePolicy` flags.

### 3. (Backup) Edit `openclaw.json` directly

Only needed when you want to set many fields at once or lock something
specific. All fields below live under `messages.tts.providers.murf`.

<details>
<summary>Example JSON config</summary>

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

All fields are optional; defaults are shown. You can set any of these with
`openclaw config set messages.tts.providers.murf.<field> <value>`.

| Field        | Type   | Default          | Description |
| ------------ | ------ | ---------------- | ----------- |
| `apiKey`     | string | *(env var)*      | Murf API key. Falls back to `MURF_API_KEY`. Never commit this. |
| `voiceId`    | string | `en-US-natalie`  | Murf voice identifier. |
| `model`      | string | `FALCON`         | Only `FALCON` is supported in this release. |
| `locale`     | string | `en-US`          | BCP-47 locale code. |
| `style`      | string | `Conversation`   | Speaking style (e.g. `Conversation`, `Newscast`). |
| `rate`       | number | `0`              | Speech rate from `-50` to `50`. |
| `pitch`      | number | `0`              | Pitch from `-50` to `50`. |
| `region`     | string | `global`         | API region. See list below. |
| `format`     | string | `MP3`            | `MP3`, `WAV`, `OGG`, or `FLAC`. Voice-note targets always use `MP3` (FALCON returns HTTP 500 for OGG). |
| `sampleRate` | number | `24000`          | Hz. One of `8000`, `16000`, `24000`, `44100`, `48000`. |

**Regions:** `au`, `ca`, `eu-central`, `global`, `in`, `jp`, `kr`, `me`,
`sa-east`, `uk`, `us-east`, `us-west`.

## Voices

List the catalog with the Murf provider selected and a valid key:

```bash
openclaw tts voices
```

Or browse them visually in the [Murf voice library](https://murf.ai/api).
A voice ID looks like `en-US-natalie`, `en-UK-harry`, `es-ES-elvira`, etc.

## Troubleshooting

| Symptom | Fix |
| ------- | --- |
| `Murf TTS is not configured` | Set `MURF_API_KEY` in your environment, or run `openclaw config set messages.tts.providers.murf.apiKey <key>`. |
| `Murf API rejected the credentials` | Key is invalid or expired. Regenerate it in the Murf dashboard. |
| Plugin not loading | `openclaw plugins list` should show `openclaw-murf-tts`. Check `openclaw plugins doctor` for errors. Restart the gateway after config edits. |
| Audio doesn't play in Telegram / WhatsApp | Voice-note channels expect Opus/OGG, but FALCON only emits MP3. The plugin still returns playable MP3 audio -- it just won't render as a native voice-note bubble. |
| Rate-limited errors | The client retries `429`/`5xx` automatically (3 attempts, exponential backoff). If it persists, check your Murf quota. |

## Development

```bash
git clone https://github.com/sanchitasunil/murf-openclaw-plugin.git
cd murf-openclaw-plugin
pnpm install
pnpm build
pnpm test
```

Live tests in `tests/live/` hit the real Murf API. They self-skip unless
both `MURF_LIVE_TEST=1` and `MURF_API_KEY` are set:

```bash
MURF_LIVE_TEST=1 MURF_API_KEY=your_key pnpm test
```