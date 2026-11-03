---
name: thunderclap-video
description: Make short-form AI videos with the Thunderclap MCP server (tools generate_video, get_video_status, resume_video, get_daily_spend). Use this whenever the user wants a video, explainer, promo, ad, teaser, reel or animation generated, wants to turn a brief, script or narration into a video, asks about a Thunderclap job (vg_... id), its status, cost, output, callback or a failed render, or wants to retry/resume one — even if they don't say "Thunderclap" or "MCP".
---

# Thunderclap video via MCP

Thunderclap turns a structured request into a finished MP4: an LLM plans shots, Higgsfield
(Kling) generates up to four sequential clips, ElevenLabs makes narration/music, ffmpeg
renders and adds any overlay text, and a signed callback announces the result. Every
accepted `generate_video` call **spends real money**, so the job of this skill is to get
the request right the first time and keep the user informed.

## Tools

| tool | use |
|---|---|
| `generate_video` | start a job; returns `job_id`, `resolved` settings, `estimated_cost_usd`, `warnings` |
| `get_video_status` | poll a job; on completion returns `output_url`, `thumbnail_url`, `actual_cost_usd`, `shot_plan` |
| `resume_video` | continue a `failed` (or stuck) job after the cause is fixed; finished clips are reused, not re-billed |
| `get_daily_spend` | today's (or a given UTC day's) spend and job counts |
| prompt `video_job_from_brief` | turns a raw brief into `generate_video` arguments |

If these tools are missing, the server isn't connected: the endpoint is
`https://<thunderclap-host>/mcp` (Streamable HTTP) with `Authorization: Bearer <VIDEO_API_KEY>`;
see "Connecting" in the skills repository README.

## Workflow

1. **Understand the ask.** Message, audience, length, aspect ratio (9:16 vertical social,
   16:9 landscape/presentations, 1:1 square), style, narration or not, music, branding.
   Ask only about things that change the result and that you can't sensibly default.
2. **Draft the arguments** using the rules below. Pick a style from
   `references/styles.md` (read it when choosing; 14 usable styles, each with its own
   strengths and defaults; avoid `screen-walkthrough`, `still-sequence`, `split-screen`).
3. **Confirm before spending** unless the user already said to go ahead: show the style,
   length, aspect, narration word count, whether there is overlay text, and a cost estimate
   (see Cost). A one-line summary is enough for small test jobs.
4. **Call `generate_video`.** Read the ack: `resolved` is what will actually happen
   (`target_length`, `clips`, `clip_seconds`, `audio_mode`, `text_overlays`,
   `callback_url`), and `warnings` explains any change (e.g. the length was resized to fit
   the narration). Tell the user about warnings in plain words.
5. **Poll `get_video_status`** every 30–60 s (a 15 s clip takes ~2–4 min; clips run one
   after another, so 60 s takes ~15–17 min). Don't poll faster; it changes nothing.
6. **Report**: status, length, actual cost, `output_url` (needs the same bearer token; files
   are deleted after 7 days by default, so the user should copy them, e.g. a delivery
   webhook that saves them to cloud storage). On failure, report `failed_step` and `error` (see Failures).

## Rules that matter

**Narration sets the length.** With a `voiceover_script`, the video is sized to the
narration: about **2.5 spoken words per second, plus 1 s**. The voiceover is generated and
measured before any clip, so a short script makes a short video instead of trailing
silence. Budget words for the length the user wants:

| length | words |
|---|---|
| 15 s | ~35 |
| 30 s | ~70 |
| 45 s | ~110 |
| 60 s | ~145 (maximum length) |

If the user's script doesn't fit the length they asked for, say so before calling ("82
words is about 33 s, not 60 s: should I expand the script or make a 34 s video?"). Never
pad or rewrite a user's script without asking; it is read verbatim. Without a script, the
planner writes one sized to `target_length`.

**Overlay text is opt-in.** Text is composited in post-production; the generated clips are
always clean. Set `on_screen_text` (≤6 strings, ≤60 chars each, shown verbatim, nothing
else added) **only when the user explicitly asks for on-screen text, captions, titles or
labels.** Do not turn scene descriptions, narration lines or labels mentioned in the visuals
into `on_screen_text`. If the user wants captions but not specific wording, use
`text_overlays: "auto"`. Omitting both gives a video with no text. Text-led styles
(`kinetic-type`, `whiteboard`, `motion-graphics` stats) look bare without text: ask whether
they want text when choosing one of those.

**Scenes follow clips.** Each clip is at most 15 s, there are at most 4, and each clip
starts from the previous clip's last frame (continuity). Write the prompt as one scene or
beat per clip (a 60 s video ≈ 4 scenes, a 30 s video ≈ 2). Packing 6 scenes into 4 clips
makes the picture lag behind the narration.

**Keep text and logos out of the generated picture.** The engine can't draw real logos or
reliable lettering. Put "readable text, brand logos, real people" in `avoid`, and use
`end_card: {"image_url": "https://…", "seconds": 1–5, "fit": "contain"}` for a logo or
call to action at the end (PNG/JPEG/WebP; 1920×1080 for 16:9 looks best).

**Idempotency.** Always send a stable, descriptive `idempotency_key`
(e.g. `acme-spring-promo-v1`). Retrying with the same key returns the original job (no new
charge, no new callback); use a new key (`-v2`) for a genuinely new version.

**Callbacks.** If `callback_url` is omitted the result goes to the operator's default
webhook (ack shows `"default"`); a user who wants the result at their own webhook must pass
`callback_url` (https). MCP never pushes results to you: poll `get_video_status`.

**Don't invent URLs.** Image, end-card and callback URLs come from the user.

## Cost and limits

- Kling standard ≈ 1.5 credits/s ≈ **$0.075 per second of video** (+ a few cents for
  narration/music). 3 s silent test ≈ $0.27; 30 s ≈ $2.30; 60 s ≈ $4.50. `pro`/`4k`
  quality and native audio cost more; `kling-3.0-turbo` is cheaper. `seedance-2.5` is
  accepted but runs on the Kling path: don't promise Seedance output.
- Limits: 2 jobs at once, 25 jobs per UTC day, $8 per job. Over a limit → 429/402
  with a reason; nothing is charged.
- `get_daily_spend` answers "how much have we spent today?".
- For a smoke test use `target_length: 3`, `audio_mode: "silent"`.

## Failures

Errors come back as normal tool results with `http_status` (validation 400, cost ceiling
402, rate limit 429, unknown job 404, not resumable 409) or as a schema error before the
API is called. Rejected requests cost nothing. Failed jobs report:

| `failed_step` | meaning / next step |
|---|---|
| `reference_image` | image URL not reachable or not PNG/JPEG/WebP; fix the URL, new job |
| `preflight` / `credit_ceiling` | Higgsfield gave no quote / too expensive; nothing billed |
| `engine` / `generation` | Higgsfield error or a clip failed twice; usually `resume_video` |
| `generation_timeout` | Higgsfield is backed up: a clip was still rendering after ~60 min. It was not resubmitted; wait a while, then `resume_video` picks up that same clip (no second charge) |
| `audio` | ElevenLabs failed (e.g. bad voice id); fix, then `resume_video` |
| `assembly` | render failed; `resume_video` |
| `higgsfield_auth` | the operator must reconnect Higgsfield; then `resume_video` |
| `plan_validation` / `planning` | planner failed; simplify the prompt, new job |

`resume_video` only works on failed or stuck jobs (completed/running → 409) and never
re-bills finished clips.

## Example

User: "Make a 30-second vertical teaser for our new budgeting app, upbeat, narration
'Money stress? Meet Penny…', end with our logo https://cdn.example.com/penny-logo.png"

```json
{
  "prompt": "Clip 1: a young professional frowns at scattered receipts and a phone full of bank alerts in a bright apartment. Clip 2: a friendly coin-shaped mascot pops out of the phone, sorts the receipts into neat stacks, and the person relaxes with a smile.",
  "style": "cartoon-explainer",
  "aspect_ratio": "9:16",
  "voiceover_script": "Money stress? Meet Penny … (≈70 words, the user's text verbatim)",
  "music_bed": true,
  "avoid": "readable text, brand logos, real people",
  "end_card": {"image_url": "https://cdn.example.com/penny-logo.png", "seconds": 3, "fit": "contain"},
  "idempotency_key": "penny-teaser-30s-v1"
}
```

No `on_screen_text` (the user didn't ask for any); length comes from the ~70-word script;
two clips, so two scenes. Estimated cost ≈ $2.30 + ~$0.10 audio.

## More detail

- `references/styles.md`: the 14 usable styles (and 3 to avoid), what each is good at, defaults, pitfalls.
- `references/fields.md`: every `generate_video` field with limits.
