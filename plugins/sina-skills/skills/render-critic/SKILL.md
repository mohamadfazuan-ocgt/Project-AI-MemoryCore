---
name: render-critic
description: "MUST use AFTER an AI video render to learn from it cheaply — or when the user says
             'learn from this render/video', 'critique the video', 'what went wrong in the render',
             'why does it look off', 'improve future renders', 'analyse the output', 'review the
             clips'. TOKEN-DISCIPLINED by design: it does NOT vision-analyse every render. It checks
             the RENDER-LESSONS cache first, vision-verifies ONLY when warranted (new scenario, a
             reported problem, or a paid final pass) using ONE contact sheet (not N clip reads),
             distils any drift into a generalised TEXT lesson in media-generation/RENDER-LESSONS.md,
             and feeds it forward to script-writer + continuity-director so future specs pre-correct
             WITHOUT re-analysis. Pay the vision cost once per lesson, never re-pay."
---

# Render Critic — Analyse Once, Learn Forever
*Vision is the expensive part. Pay it once per lesson, turn it into text, and never re-pay.*

> Invocable as `/render-critic`. The **post-render learning loop** that closes the pipeline:
> `script-writer` → `continuity-director` → `storyboard-runner` (render) → **`render-critic` (learn)** ↺.
> Its output (`media-generation/RENDER-LESSONS.md`) feeds back into the first two skills.

## Activation

When this skill activates, output:

`🔎 Render Critic — cache-first, vision-last: check lessons → (only if needed) one contact sheet → distil a text lesson → feed forward. Analyse once, never re-pay.`

Then execute the protocol.

## Context Guard

| Context | Status |
|---------|--------|
| **A render just finished + user wants to learn / improve** | ACTIVE — run the cost gate first |
| **"learn from this render", "critique the video", "why does it look off"** | ACTIVE |
| **A paid final / HD pass before delivery** | ACTIVE — verify once, it's worth the tokens |
| **Render scenario already covered by RENDER-LESSONS.md + no reported issue** | **DORMANT — skip vision, trust the cache (the whole point)** |
| **Pre-render continuity** | DORMANT — that's `continuity-director` |
| **Writing scenes** | DORMANT — that's `script-writer` |

## Cost Discipline (why this skill exists)
Analysing every generated video with vision burns tokens on every render. This skill makes that a
**one-time cost per distinct lesson**, not a per-render tax:
1. **Cache-first.** Read `RENDER-LESSONS.md` (cheap text). If the render's risks are already covered and the user reports no problem → **do not vision-analyse**. Confirm the lessons were applied and stop.
2. **Vision only when it pays.** Verify with pixels only on: (a) a **new scenario** not in the cache, (b) a **reported problem**, or (c) a **paid final/HD pass**.
3. **One image, not N.** When you do verify, build **one contact sheet** (midframes tiled into a single image) — never read each clip separately.
4. **Text is the cache.** Every finding becomes a generalised rule in `RENDER-LESSONS.md`. After that, downstream skills pre-correct from text — zero vision.

## Protocol

### Step 1: Cost gate (decide if vision is even needed)
- [ ] Read `media-generation/RENDER-LESSONS.md`.
- [ ] Cross-check the render's spec against the lessons + Pre-flight checklist.
- [ ] **If covered + no reported issue → STOP.** Report "lessons already cover this; rendered without re-analysis." (This is a success, not a skip.)
- [ ] Else → proceed to verify (state *why*: new scenario / reported issue / final pass).

### Step 2: Cheap verify (ONE contact sheet)
- [ ] Extract one midframe per clip and tile into a single image (proven command below), then read **that one image**.
```bash
d="<render-dir>"; s="<scratchpad>"
for i in $(ls "$d"/clips/*.mp4); do ffmpeg -nostdin -v error -ss 2.5 -i "$i" -frames:v 1 "$s/f$(basename "$i" .mp4).png" -y; done
ffmpeg -nostdin -v error -framerate 1 -pattern_type glob -i "$s/f*.png" -vf "scale=380:-1,tile=2x4:padding=6" -frames:v 1 "$s/contact.png" -y
```
- [ ] (Run the loop under `bash`, not zsh — zsh aborts on empty globs.)

### Step 3: Diagnose against the six continuity domains
- [ ] Per beat, note drift in: environment, lighting/colour-temp, props, physics, camera/spatial — plus model artifacts (bad hands, garbled text, morphing, identity slip).
- [ ] Tie each issue to the **prompt/spec line that caused it** (so the lesson is actionable).

### Step 4: Distil → generalised lessons (not one-offs)
- [ ] Convert each finding to a reusable rule: **[trigger pattern] → [observed failure] → [prompt fix]**.
- [ ] Bad (one-off): "beat 4 was night." Good (general): "city/map prompts without a stated time-of-day → model defaults to night → always state time-of-day; use a holo overlay with 'no sky' for UI shots."

### Step 5: Store (dedupe + count)
- [ ] Append to `RENDER-LESSONS.md`. If the lesson exists → **increment `seen`**, don't duplicate.
- [ ] **3+ `seen` → promote** to the auto-applied Pre-flight checklist.
- [ ] For a major/systemic lesson, optionally drop a `post-mortem` entry too (house learning system).

### Step 6: Feed forward + report
- [ ] State what was learned and that `script-writer` + `continuity-director` will now pre-correct it for free.
- [ ] Quantify the win: "this lesson is now text — future renders skip the vision pass for it."

## Mandatory Rules
1. **Cache-first, vision-last.** Never auto-vision a render. Read the lessons + run the cost gate first; skipping vision because the cache covers it is the goal, not a failure.
2. **One contact sheet, never N clip reads.** Cheap verify only.
3. **Generalise.** A lesson must apply to *future* jobs — pattern → failure → fix, not "this clip was bad."
4. **Dedupe + count + promote.** Same lesson again = increment `seen`; 3+ becomes a hard pre-flight rule.
5. **Text over pixels.** The durable store is `RENDER-LESSONS.md`. Never rely on keeping the video file around.
6. **Model behaviour only.** Tooling bugs (ARG_MAX, bash 3.2, dotenv) belong in the workflow memory / the helper — not in render lessons.
7. **Feed forward, don't re-render.** This skill learns; re-rendering a fixed beat is `storyboard-runner`'s job (offer it, ~$0.25/beat).
8. **Fail loud.** Render dir/clips missing, or ffmpeg/ffprobe unavailable → say so; don't fabricate a critique.

## Edge Cases
| Situation | Behavior |
|-----------|----------|
| User reports a specific problem ("scene 4 looks like night") | Vision-verify just that, but still distil a *general* lesson, not a one-off note. |
| Render fully covered by existing lessons | Skip vision; confirm lessons applied; report the tokens saved. |
| Same lesson keeps recurring (3+) | Promote to Pre-flight checklist; flag that the upstream skill should bake it in permanently. |
| No clips / wrong path | Fail loud; ask for the render directory. |
| User wants every render analysed regardless | Honour it, but warn it re-pays vision tokens the cache is meant to save; suggest reserving it for final passes. |

## Level History
- **Lv.1** — Base: cost-aware post-render learning loop. Cache-first gate (skip vision when `RENDER-LESSONS.md` already covers the scenario); cheap one-contact-sheet verify when warranted; diagnose across the six continuity domains + model artifacts; distil generalised `[trigger]→[failure]→[fix]` lessons; dedupe/count/promote to an auto-applied Pre-flight checklist; feed forward to `script-writer` + `continuity-director`. (Origin: user — "analysing all the generated video will consume token, I want Sina learn from it", 2026-06-29. Built so the vision cost is paid once per lesson and amortised across all future renders.)
