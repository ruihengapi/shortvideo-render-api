# ShortVideo Render API

Turn a script plus your images or clips into a **publish-ready vertical video** with one HTTP call.
You get back a 1080x1920 (9:16) MP4 with AI voice over, sentence-accurate burned-in subtitles,
background music and audio normalised to **-14 LUFS** — no editing timeline, no ffmpeg, no caption sync.

- Docs: <https://api.ruihengit.com/vid/docs>
- Pricing and live demo: <https://api.ruihengit.com/vid/en>
- OpenAPI: <https://api.ruihengit.com/vid/openapi.json>
- Marketplace listing: <https://rapidapi.com/ruihengapi/api/shortvideo-render>

## Why

Faceless channel operators, no-code builders and marketing teams all need the same thing: many vertical
videos per day, each with voice over and readable subtitles. Doing it yourself means a TTS call, a subtitle
aligner, an ffmpeg pipeline, a loudness pass and a storage layer — five moving parts and a queue to babysit.
This API replaces all five with one request, billed per output minute.

## Quick start

```bash
curl -sS -X POST "https://api.ruihengit.com/vid/v1/videos" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "spec": {
      "scenes": [
        {"id": "s0", "type": "color", "color": "#101418"},
        {"id": "s1", "type": "image", "src": "https://example.com/hero.jpg"}
      ],
      "lines": [
        {"text": "One sentence in, one finished video out.", "scene": "s0"},
        {"text": "Voice over, subtitles and music are already done.", "scene": "s1"}
      ]
    }
  }'
```

```json
{"job_id": "job_8f21...", "status": "queued", "minutes_reserved": 1}
```

Poll the job, then download the MP4:

```bash
curl -sS "https://api.ruihengit.com/vid/v1/videos/job_8f21..." -H "X-API-Key: $API_KEY"
curl -sSL "https://api.ruihengit.com/vid/v1/videos/job_8f21.../file" -H "X-API-Key: $API_KEY" -o out.mp4
```

Pass `callback_url` in the request body to receive the finished payload instead of polling.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/v1/videos` | Queue a render from a script spec |
| GET | `/v1/videos` | List recent jobs for your key |
| GET | `/v1/videos/{id}` | Job status, timings, output metadata |
| GET | `/v1/videos/{id}/file` | Download the finished MP4 |
| DELETE | `/v1/videos/{id}` | Cancel a queued job, release its minutes |
| GET | `/v1/usage` | Remaining minutes, jobs today, plan limits |

## Output

| Property | Value |
|---|---|
| Resolution | 1080x1920 (9:16), 720p and square variants available |
| Audio | Voice over + ducked background music, -14 LUFS |
| Subtitles | Burned in, timed per sentence |
| Voices | 39 voices across 24 languages |
| Delivery | Async job, polling or webhook, direct MP4 link |

## Pricing

| Plan | Price | Output minutes | Roughly |
|---|---|---|---|
| Basic | $0 | free evaluation quota | try the endpoints |
| Starter | $9 / month | 100 | ~200 videos of 30 s |
| Growth | $29 / month | 500 | ~1,000 videos of 30 s |
| Scale | $79 / month | 2,000 | ~4,000 videos of 30 s |

Unused minutes roll over while the subscription is active. Failed renders never consume minutes.

Buy direct: [Starter](https://payhip.com/b/7amGH) · [Growth](https://payhip.com/b/kUgX4) ·
[Scale](https://payhip.com/b/Td8fS) — or subscribe on
[RapidAPI](https://rapidapi.com/ruihengapi/api/shortvideo-render).

## Errors

| Code | Meaning | What to do |
|---|---|---|
| 401 | Missing or invalid key | Check `X-API-Key` |
| 402 | Not enough minutes | Top up or upgrade |
| 422 | Spec rejected | The message names the scene or line at fault |
| 429 | Rate limited | Back off and retry |

## Notes

- Landscape sources are fitted rather than cropped, so nothing important leaves the frame.
- Scene types: `image`, `video`, `color`; each scene can carry its own duration.
- The same pool serves the RapidAPI gateway, so behaviour is identical on both rails.

## License

Documentation and examples in this repository: MIT. The hosted API itself is a commercial service.
