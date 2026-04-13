# openclaw-murf-tts

> **Community-managed plugin.** Maintained by [@sanchitasunil](https://github.com/sanchitasunil)
> from a personal GitHub profile, not by the OpenClaw core team. File issues
> and pull requests on this repo.

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

1. Get a Murf API key from the [Murf API dashboard](https://murf.ai/api/dashboard).
2. Set the key in your environment:
   ```bash
   export MURF_API_KEY="your_key_here"
   ```
3. Add the provider to your OpenClaw config (`openclaw.json` or `openclaw.json5`):
   ```json
   {
     "messages": {
       "tts": {
         "provider": "murf",
         "providers": {
           "murf": {}
         }
       }
     },
     "plugins": {
       "enabled": true,
       "entries": {
         "murf-tts": { "enabled": true }
       }
     }
   }
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

With an empty `murf: {}` block, the plugin uses sensible defaults and reads
the API key from `MURF_API_KEY`. See `openclaw.config.example.json5` for
a fully annotated example.

## Configuration

All fields go under `messages.tts.providers.murf`. Every field is optional;
defaults are shown below.

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

## In-message directives

When OpenClaw allows directive overrides, you can tweak Murf parameters
inline per message:

```
@tts voiceid=en-US-jackson style=Newscast rate=10 pitch=-5
```

Supported keys: `voiceid`, `model`, `style`, `rate`, `pitch`, `locale`,
`format`. Each key respects your OpenClaw `SpeechModelOverridePolicy` flags.

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
| `Murf TTS is not configured` | Set `MURF_API_KEY` in your environment, or add `apiKey` under `messages.tts.providers.murf`. |
| `Murf API rejected the credentials` | Key is invalid or expired. Regenerate it in the Murf dashboard. |
| Plugin not loading | `openclaw plugins list` should show `murf-tts`. Check `openclaw plugins doctor` for errors. Restart the gateway after config edits. |
| Audio doesn't play in Telegram / WhatsApp | Voice-note channels expect Opus/OGG, but FALCON only emits MP3. The plugin still returns playable MP3 audio -- it just won't render as a native voice-note bubble. |
| Rate-limited errors | The client retries `429`/`5xx` automatically (3 attempts, exponential backoff). If it persists, check your Murf quota. |

## Development

```bash
git clone https://github.com/sanchitasunil/openclaw-murf-tts.git
cd openclaw-murf-tts
pnpm install
pnpm build
pnpm test
```

Live tests in `tests/live/` hit the real Murf API. They self-skip unless
both `MURF_LIVE_TEST=1` and `MURF_API_KEY` are set:

```bash
MURF_LIVE_TEST=1 MURF_API_KEY=your_key pnpm test
```

## License

MIT. See [LICENSE](./LICENSE).
