# `generate_video` fields

Generated from the Thunderclap API's request model (`app/api_docs.py` in the service repo);
the MCP tool takes the same
fields. Only `prompt` and `style` are required.

| field | type | description |
|---|---|---|
| `prompt` **required** | str (min_length=10, max_length=2000) | What the video is about. |
| `style` **required** | Literal['cartoon-explainer', '2d-character', 'whiteboard', 'motion-graphics', 'kinetic-type', 'stop-motion', 'live-action', 'still-sequence', 'screen-walkthrough', 'avatar-presenter', 'mixed-media', 'testimonial', '3d-animation', 'live-action-motion', 'day-in-life', 'split-screen', 'music-visual'] | Visual style; also sets defaults for length and audio. `screen-walkthrough` is rejected (caller footage is not supported in v1); `still-sequence` and `split-screen` generation is not implemented yet. |
| `target_length` | int (ge=3, le=60) | Seconds. ≤15 → 1 clip, 16–30 → 2, 31–45 → 3, 46–60 → 4. Default: the style's length. With voiceover the narration decides the final length (see `voiceover_script`). |
| `aspect_ratio` | Optional[Literal['9:16', '1:1', '16:9']] | Default `9:16`. |
| `engine` | Optional[Literal['kling-3.0', 'kling-3.0-turbo', 'seedance-2.5']] | Default `kling-3.0`. `seedance-2.5` is accepted but runs on the Kling path. |
| `quality` | Optional[Literal['standard', 'pro', '4k']] | Default `standard`; `pro`/`4k` cost more. |
| `audio_mode` | Optional[Literal['native', 'voiceover', 'music', 'silent']] | `native`: sound generated with the clip (single-clip only; multi-clip jobs switch to `voiceover`, reported as `audio_mode_forced`). `voiceover`: ElevenLabs reads a script. `music`: ElevenLabs backing track. `silent`: no audio track. Default: the style's mode. |
| `voiceover_script` | str (max_length=1500) | Used verbatim and forces `audio_mode=voiceover`. Default: written by the planner. Sets the video length: the voiceover is generated first and the clips are sized to it (narration + 1 s, ~2.5 words/s); a mismatch with `target_length` is reported in the ack's `warnings`. |
| `voice` | str (pattern=^[A-Za-z0-9]{8,64}$) | ElevenLabs voice_id. Default `DtQLDxHbTiQTpVVanTy1`. |
| `on_screen_text` | list[str] (max_length=6) | Up to 6 captions (≤60 chars each); the planner times them. With `text_overlays` omitted, the video shows exactly these strings, verbatim, and nothing else. |
| `reference_image` | str | https URL of a start image for clip 1 (image-to-video). |
| `avoid` | str (max_length=300) | Things to keep out, folded into the prompts. |
| `callback_url` | str | https URL that receives the signed `video.completed` / `video.failed` POST. Omitted: the operator's `DEFAULT_CALLBACK_URL` is used if set (the ack shows `resolved.callback_url: "default"`, never the URL); otherwise no callback is sent. |
| `metadata` | dict[str, Any] | Any JSON object ≤4 KB, echoed in callbacks. |
| `shot_plan` | dict[str, Any] | Your own plan (skips the LLM planner): `{"clips": [{"index": 0, "duration": 5, "shots": [{"prompt": "…", "duration": 5}], "overlay_text": [{"text": "…", "start": 0, "end": 2}]}], "voiceover_script": "", "music_brief": ""}`. |
| `idempotency_key` | str | Same key → the original job comes back (200, no new charge). |
| `music_bed` | bool | Voiceover only: also generate an instrumental bed (the planner writes the music brief) that ducks under the narration. Default false. Adds ElevenLabs music cost. |
| `end_card` | EndCard | A still image appended after the generated clips (logo / call to action). The narration and music continue under it; the output gets this many seconds longer than target_length. |
| `text_overlays` | Optional[Literal['none', 'exact', 'auto']] | `none`: no overlay text at all (cannot be combined with `on_screen_text`). `exact`: only `on_screen_text`, verbatim (requires it). `auto`: the planner may add its own captions, plus `on_screen_text` if given. Default: `exact` when `on_screen_text` is sent, otherwise `none`: overlay text is opt-in for every style. Enforced at render time. A style's legal label (e.g. "Dramatization") is separate and unaffected. |

## `end_card`

| field | type | description |
|---|---|---|
| `image_url` | str | https URL of a PNG/JPG. Best at the output aspect ratio (1080x1920 for 9:16, 1920x1080 for 16:9). |
| `seconds` | float | How long the card holds. Default 3. |
| `fit` | Literal['contain', 'cover'] | `contain`: whole image on black bars (never crops). `cover`: fill the frame, center-cropped. |

## Ack (`generate_video` result)

`job_id`, `status` (`queued`), `resolved` {style, build_mode, engine, quality, target_length,
aspect_ratio, audio_mode, voice, clips, clip_seconds, audio_mode_forced, text_overlays,
callback_url (`"default"` = operator webhook, `null` = none)}, `estimated_cost_usd`,
`ignored_fields` (unknown fields that were dropped), `warnings`, `status_url`.
A replayed `idempotency_key` returns the original job with `idempotent_replay: true`.

## Status (`get_video_status` result)

`status` moves `queued → planning → generating → audio → assembling → delivering →
completed` (or `failed`). Completed jobs include `output_url`, `thumbnail_url`,
`actual_length`, `actual_cost_usd`, `completed_at`, `shot_plan`; failed jobs `failed_step`
and `error`.
