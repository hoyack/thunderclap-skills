# Thunderclap styles

Source of truth: the Thunderclap service's style catalog (`seeds/styles.json` in the service
repo); this file is generated from it. Refresh when styles change.
Pass the **slug** as `style`. **14 are usable.** Don't use these three:

- `screen-walkthrough`: disabled (needs real screen recordings); rejected.
- `still-sequence` and `split-screen`: rejected because their generation is **not
  implemented**. Use `cartoon-explainer`/`motion-graphics` instead, or describe a
  before/after across two clips.

Defaults below apply when the request doesn't set them. With `voiceover_script` the
narration sets the length; overlay text is opt-in for every style. Planner instructions
below that mention `overlay_text` only take effect when the request turns text on
(`on_screen_text` or `text_overlays: "auto"`); otherwise the planner is told to use none.

| slug | good for | default audio | default length |
|---|---|---|---|
| `cartoon-explainer` | Explaining one idea simply; product/service explainers. | voiceover | 15 s |
| `2d-character` | A recurring character telling a short story. | native | 15 s |
| `whiteboard` | Hand-drawn process explanations. | voiceover | 15 s |
| `motion-graphics` | Abstract animated backgrounds for stats, concepts, corporate. | voiceover | 15 s |
| `kinetic-type` | Text-driven promos where the words ARE the video. | music | 12 s |
| `stop-motion` | Charming handcrafted clay/object animation. | music | 12 s |
| `live-action` | Realistic dramatized scenes with people, ad-style. | native | 15 s |
| `still-sequence` | (not implemented: avoid) | voiceover | 15 s |
| `screen-walkthrough` | (disabled) | voiceover | 30 s |
| `avatar-presenter` | One presenter talking to camera. | native | 15 s |
| `mixed-media` | Real footage with animated elements reacting. | native | 15 s |
| `testimonial` | First-person 'customer story' style. | native | 15 s |
| `3d-animation` | Original 3D mascots and characters. | native | 15 s |
| `live-action-motion` | Live footage with graphic callouts/stats. | voiceover | 15 s |
| `day-in-life` | Candid behind-the-scenes sequences. | native | 20 s |
| `split-screen` | (not implemented: avoid) | music | 12 s |
| `music-visual` | Music-only product/mood pieces. | music | 15 s |

## `cartoon-explainer`: Cartoon explainer

- **Good for:** Explaining one idea simply; product/service explainers.
- **Defaults:** audio `voiceover`, 15 s, engine `kling-3.0`, multi-shot clips.
- **Notes:** Narration-led. The default choice for explainers.
- **What the planner is told:** Explain one idea simply. Friendly, upbeat narration carries the story; visuals illustrate each beat. Key words go to overlay_text, never into the video prompt.

## `2d-character`: 2D character animation

- **Good for:** A recurring character telling a short story.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`.
- **Notes:** Native lip-sync dialogue works on single-clip videos (≤15 s); multi-clip switches to voiceover. Pass a `reference_image` to keep the character consistent.
- **What the planner is told:** One or two recurring characters drive the story. If a reference image is supplied, describe that character identically in every shot. Dialogue is spoken by the character (native lip-sync).

## `whiteboard`: Whiteboard animation

- **Good for:** Hand-drawn process explanations.
- **Defaults:** audio `voiceover`, 15 s, engine `kling-3.0`.
- **Notes:** Labels/terms only appear as overlay text: ask whether the user wants `on_screen_text` or `text_overlays: "auto"`, otherwise it is drawings + narration only.
- **What the planner is told:** Visuals are simple drawings being sketched; NEVER ask the engine to draw words or labels. All labels, terms and numbers go to overlay_text timed to the narration.

## `motion-graphics`: Motion graphics / infographic

- **Good for:** Abstract animated backgrounds for stats, concepts, corporate.
- **Defaults:** audio `voiceover`, 15 s, engine `kling-3.0`.
- **Notes:** The engine makes backgrounds only; numbers/charts need overlay text (`on_screen_text` with exact values). Without text it is mood visuals under narration.
- **What the planner is told:** The engine makes abstract animated backgrounds only. Every number, chart label and statistic goes to overlay_text with exact values; they must come out exactly as written.

## `kinetic-type`: Kinetic typography

- **Good for:** Text-driven promos where the words ARE the video.
- **Defaults:** audio `music`, 12 s, engine `kling-3.0`.
- **Notes:** Needs overlay text to make sense: use `on_screen_text` (exact phrases) or `text_overlays: "auto"`. Default audio is music.
- **What the planner is told:** The words ARE the video. Break the message into short punchy phrases in overlay_text with precise timings (one phrase per beat). The video prompt is only a mood background.

## `stop-motion`: Stop-motion

- **Good for:** Charming handcrafted clay/object animation.
- **Defaults:** audio `music`, 12 s, engine `kling-3.0`.
- **Notes:** Default audio is music; no dialogue unless asked.
- **What the planner is told:** Charming handcrafted feel. Favor physical gags and transformations that suit clay. Music plus foley-style sound; no dialogue unless the caller asks.

## `live-action`: Live-action dramatized

- **Good for:** Realistic dramatized scenes with people, ad-style.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`, multi-shot clips.
- **Notes:** Real-looking people: avoid resembling real persons. Native dialogue only on single-clip videos.
- **What the planner is told:** Real-world scenes with real-looking people. Natural dialogue with lip-sync (max ~2.5 words/second). Use multi-shot cuts for pacing like a real ad.

## `still-sequence`: Still-sequence

- **Good for:** Narrated slideshow of stills with slow pan/zoom (Ken Burns).
- **Defaults:** audio `voiceover`, 15 s, engine `kling-3.0`.
- **Notes:** NOT IMPLEMENTED: generation makes normal clips; the Ken Burns stills layout is not wired up. Avoid.
- **What the planner is told:** Each shot is ONE still image; assembly adds slow pan/zoom (Ken Burns). Write image prompts, not motion prompts. Narration carries the story across the stills.

## `screen-walkthrough`: Screen-record walkthrough

- **Good for:** (disabled)
- **Defaults:** audio `voiceover`, 30 s, engine `kling-3.0`.
- **Notes:** Needs real screen recordings; not supported in v1. Requests are rejected.
- **What the planner is told:** Requires caller-supplied screen footage; an engine cannot show a real product accurately. Disabled in v1.

## `avatar-presenter`: Avatar presenter

- **Good for:** One presenter talking to camera.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`.
- **Notes:** Use a `reference_image` for a consistent presenter. Native dialogue on single-clip videos; longer videos use voiceover.
- **What the planner is told:** One presenter addresses the viewer for the whole video. Keep the same framing between clips. If a reference image is supplied the presenter must match it in every shot. Dialogue is the script.

## `mixed-media`: Mixed real-life + animation

- **Good for:** Real footage with animated elements reacting.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`.
- **Notes:** Results vary; keep shots simple and concrete. Expect re-runs.
- **What the planner is told:** Real footage is the base; animated elements react to it (arrows, creatures, effects). Results vary; keep shots simple and concrete. Expect re-runs.

## `testimonial`: Testimonial-style dramatized

- **Good for:** First-person 'customer story' style.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`.
- **Notes:** Assembly adds a "Dramatization" label (legal); never present the speaker as a real customer.
- **What the planner is told:** A relatable person shares an experience in first person. Conversational dialogue with lip-sync. The Dramatization label is applied in assembly; never present the speaker as a real customer.

## `3d-animation`: 3D animation

- **Good for:** Original 3D mascots and characters.
- **Defaults:** audio `native`, 15 s, engine `kling-3.0`.
- **Notes:** Original characters only: nothing resembling an existing brand mascot.
- **What the planner is told:** Home of talking mascots: invent ORIGINAL characters only; never resemble an existing brand mascot (no geckos for insurance, etc.). Character dialogue with lip-sync.

## `live-action-motion`: Live-action + motion combo

- **Good for:** Live footage with graphic callouts/stats.
- **Defaults:** audio `voiceover`, 15 s, engine `kling-3.0`.
- **Notes:** Callouts are overlay text: ask for `on_screen_text` if the user wants specific callouts.
- **What the planner is told:** Compose shots with room for graphics (upper third or side). Callouts, labels and stats go to overlay_text; the graphic layer is added in assembly.

## `day-in-life`: Behind-the-scenes / day in the life

- **Good for:** Candid behind-the-scenes sequences.
- **Defaults:** audio `native`, 20 s, engine `kling-3.0`, multi-shot clips.
- **Notes:** Multi-shot cuts (2–4 quick scenes per clip) are the style; ambient native sound.
- **What the planner is told:** A sequence of candid moments across a day. Multi-shot cuts are the style: 2-4 quick scenes per clip. Ambient native sound; optional voiceover narrating over it.

## `split-screen`: Split-screen comparison

- **Good for:** Before/after or problem/solution comparisons.
- **Defaults:** audio `music`, 12 s, engine `kling-3.0`, cost ×2.
- **Notes:** NOT IMPLEMENTED: the left/right pair generation is not wired up (it would be ~2× cost). Avoid; show before/after across two clips instead.
- **What the planner is told:** Plan PAIRS of shots: left = before/problem, right = after/solution, same timing. Each side is its own generation; assembly places them side by side. Labels for each side go to overlay_text.

## `music-visual`: Music-only visual demo

- **Good for:** Music-only product/mood pieces.
- **Defaults:** audio `music`, 15 s, engine `kling-3.0`.
- **Notes:** No dialogue; cuts every 2–3 s to the beat.
- **What the planner is told:** No dialogue anywhere. Cut to an imagined beat every 2-3 seconds; describe motion that syncs to music. Provide a music_brief matching the product mood.
