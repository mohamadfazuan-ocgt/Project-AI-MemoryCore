# Character-Consistent Storyboard Prompt Kit

A model-agnostic prompt template for generating **consistent characters**, **deliberate camera
work**, and **storyboards** that feed cleanly into AI video (image-to-video). Works with any image
model — Midjourney, Gemini/"Nano Banana", Stable Diffusion, Seedream, DALL·E, and **Seedance** (this repo's renderer).

Copy the blocks below, fill the `[SLOTS]`, paste into your image model. Share this file freely.

---

## The workflow (why it's 3 stages)

```
STAGE 1                 STAGE 2                      STAGE 3
Character Sheet   ─────► Storyboard (N panels)  ────► Video (image-to-video)
(lock identity)         (beats + camera per panel)   (animate one beat per clip)

   the "bible"          re-uses the bible verbatim    feed sheet as reference,
   + a turnaround       + adds SHOT / ANGLE / MOVE    panel as first frame
```

You can't get consistency by re-describing the character loosely each time. You lock it **once** in a
character bible, then **repeat that exact text** in every storyboard panel and every video prompt. The
turnaround sheet from Stage 1 becomes a *reference image* you re-feed into later stages.

---

## The 5 golden rules of consistency

1. **Write a Character Bible once.** Paste it **verbatim** into every panel and every clip. Never
   paraphrase it — even small wording changes drift the face/outfit.
2. **Lock the seed.** Re-use the same seed number across panels (where the model supports it) so the
   render is stable. Vary only the camera/action.
3. **Feed a reference image.** Generate the turnaround sheet first, then pass it back into every
   subsequent generation (`--cref`, image input, IP-Adapter — see model knobs below).
4. **One style lock.** Pick one medium + lighting + palette and repeat it in every prompt. Mixed
   styles = a different-looking character.
5. **Change only TWO things per panel: the action and the camera.** Identity, wardrobe, style, palette
   stay frozen.

---

## Stage 1 — Character Turnaround Sheet (the identity lock)

> Goal: one image, same character shown front / three-quarter / side / back, so later prompts (and
> video models) have a full reference of the face and outfit.

```
A character turnaround / model sheet of [CHARACTER NAME], [AGE + BUILD], shown in FOUR views on one
sheet, left to right: front view, 3/4 front view, side profile, back view. The SAME character, with
an IDENTICAL face, body, and outfit in every view, neutral A-pose, evenly lit on a plain [COLOR]
background.

CHARACTER BIBLE (keep identical in every later prompt):
- Face: [face shape, skin tone, eyes, eyebrows, nose, facial hair, glasses?, distinguishing marks]
- Hair / headwear: [exact style + color, or exact head covering]
- Wardrobe: [garment names + EXACT colors + fabric + pattern, top to bottom]
- Props / weapon: [held items, where carried]
- Body language: [default posture / vibe]

STYLE LOCK:
- Medium: [e.g. cinematic 3D render / cel anime / graphic-novel ink / painterly]
- Lighting: [e.g. soft even studio light]
- Palette: [3–4 dominant colors]

Labels and clean spacing between views. Consistent proportions across all four. [ASPECT RATIO].
```

**Worked example (fill-in shown filled):**

```
A character turnaround / model sheet of ABDUL, an elder Malay silat grandmaster (60s, lean, upright),
shown in FOUR views on one sheet: front, 3/4 front, side profile, back. The SAME character with an
IDENTICAL face and outfit in every view, neutral A-pose, evenly lit on a plain warm-grey background.

CHARACTER BIBLE:
- Face: weathered, warm brown skin, calm focused eyes, thin-rimmed glasses, short grey-flecked beard.
- Headwear: black tengkolok headwrap, "Dendam Tak Sudah" fold.
- Wardrobe: deep indigo songket baju (long sleeves) with gold brocade trim; checkered blue-and-gold
  samping sash at the waist; dark trousers; barefoot.
- Props: antique wavy royal keris (kris dagger) tucked at the sash.
- Body language: composed, grounded, dignified.

STYLE LOCK:
- Medium: cinematic 3D render, stylized realism (NOT photoreal).
- Lighting: soft even studio light.
- Palette: indigo, gold, warm brown, parchment.

Labels under each view, clean spacing, consistent proportions. 4:3.
```

> ⚠️ **Make it stylized, not photoreal.** A photoreal human face gets your sheet flagged as a "real
> person" and **rejected as input by most video models** (anti-deepfake filters — this is exactly what
> blocked our first render). "Cinematic 3D render / stylized realism / anime / painterly" sails
> through *and* still looks great. This single word choice decides whether Stage 3 works.

---

## Stage 2 — Storyboard (beats + camera)

> Goal: N panels telling the sequence. Each panel re-uses the bible and adds a **shot size + angle +
> lens + camera move + action + expression**. The camera move is what you'll carry into the video
> prompt later.

**Header (paste once, top of the prompt):**

```
A [N]-panel storyboard, [N] equal panels in a grid, numbered, each with a one-line caption.
Same character and style in every panel (see bible). [ASPECT RATIO per panel]. [STYLE LOCK from Stage 1].

CHARACTER BIBLE: [paste the exact bible block from Stage 1]
```

**Per-panel line (repeat for each panel):**

```
PANEL [n] — SHOT: [shot size] | ANGLE: [camera angle] | LENS: [lens] | MOVE: [camera movement] |
ACTION: [what the character does] | EXPRESSION: [face/emotion] | CAPTION: "[short caption]"
```

**Worked example (3 of 8 panels):**

```
PANEL 1 — SHOT: full shot | ANGLE: eye-level | LENS: 35mm | MOVE: slow push-in |
ACTION: settles into a grounded ready stance, recentering | EXPRESSION: calm, breathing |
CAPTION: "Re-centering."

PANEL 5 — SHOT: medium shot | ANGLE: low angle | LENS: 50mm | MOVE: whip-pan follow |
ACTION: explosive keris thrust, crisp body snap | EXPRESSION: fierce focus |
CAPTION: "Explosive strike."

PANEL 6 — SHOT: wide shot | ANGLE: low angle | LENS: 24mm | MOVE: arcing crane up |
ACTION: dynamic aerial backflip, sash trailing | EXPRESSION: controlled | CAPTION: "Aerial transition."
```

---

## Stage 3 — Video (image-to-video) handoff

> Goal: animate. Best fidelity = **one beat (one panel) per clip**, not all 8 in one render.

```
[Paste the Character Bible verbatim].
MOTION: [describe the single beat as continuous motion].
CAMERA: [the MOVE from that panel — push-in / orbit / crane / tracking / handheld] + [pacing: calm vs explosive].
[duration] seconds, [resolution], [aspect ratio], cinematic motion blur.
```

**What to feed the video model:**

| Input slot | What to put | Why |
|------------|-------------|-----|
| `reference image` | the **turnaround sheet** | locks the character's look across the clip |
| `first frame` | the **panel image** for this beat (a clean single frame) | stable, predictable start pose |
| `last frame` *(optional)* | the next beat's panel | guides where the motion ends |
| prompt | bible + motion + camera | the choreography + identity in words |

**Hard-won gotchas:**
- **Photoreal faces are rejected** as video input by many providers (real-person filter). Stylize at
  Stage 1. (Being AI-generated does **not** exempt you — the filter judges the pixels, not the origin.)
- **One beat per clip.** Cramming 8 beats into one 5–10s render = blurry, mushy motion.
- **Keep aspect ratio fixed** across sheet → storyboard → video, or the framing jumps.
- **Carry the camera move forward.** The panel's `MOVE` becomes the video's `CAMERA` line.

---

## The "doesn't look AI" preset (cinematic motion) ⭐

> Goal: the single biggest quality jump for image-to-video. This is the look from
> `jungle-bird-race-v2` (two macaws racing through a jungle) — fast, weighty, and reading as **real
> footage, not AI**. Bolt these lines onto any Stage 3 clip.

**Core principle:** the sense of speed comes from the **background rushing past + motion blur**, *not*
from speeding the subject up in post. Lock the camera to the subject and let the world streak by. No
`setpts`/ffmpeg speed-up needed — the velocity is baked into the prompt (verified: those clips are
native 5s @ 24fps, plain concat, zero post speed ramp).

**Paste this in place of the plain Stage 3 `CAMERA:` line:**

```
CAMERA: tracking-follow locked to the subject — camera flies alongside at the subject's speed; subject
stays sharp and centered while the background streaks past (parallax). Explosive pacing. Shallow depth
of field: subject sharp, background soft. Cinematic motion blur (directional, per-frame).
LIGHT: volumetric god-rays through the scene + atmospheric haze for depth layering.
```

**Why each lever kills the "AI look" (ranked by impact):**
1. **Camera tracks WITH the subject** — the #1 tell. AI-slop = static camera + a subject that floats.
   Real footage = the camera *chases*, so the subject is steady and the background smears with parallax.
2. **Cinematic motion blur** — directional per-frame blur reads as a real shutter; perfectly crisp frames read as CGI/AI.
3. **Speed via background velocity, not post** — fast subject motion + fast background scroll + explosive pacing. Native and reproducible; no ffmpeg ramp.
4. **Stylized, never photoreal** — see the Stage 1 warning; stylization hides the artifacts photoreal exposes.
5. **Volumetric god-rays + haze** — foreground / mid-light / background depth layers kill the flat look.
6. **Shallow DOF** — subject sharp, background soft = it reads as a lens, not a render.
7. **One clear action beat** (bank / dive / dodge) — gives weight and intent, not drift.

**Worked example (the macaw race, beat 02):**

```
[Character Bible: the two macaws — blue-and-gold + scarlet, exact plumage...].
MOTION: the two macaws rocket side by side through the jungle, banking hard around trunks, beaks open mid-call.
CAMERA: tracking-follow locked to the birds — camera flies alongside at their speed; birds sharp and
centered, jungle canopy streaking past in parallax. Explosive pacing. Shallow DOF. Cinematic motion blur.
LIGHT: god-rays breaking through the canopy + soft atmospheric haze.
5 seconds, 864x496, 16:9.
```

---

## Camera cheat-sheet (shared by stills + video)

**Shot size:** ECU (extreme close-up) · CU (close-up) · MCU (medium close-up) · MS (medium) ·
MLS (medium-long) · FS (full / full body) · LS (long) · WS / EWS (wide / extreme wide).

**Angle:** eye-level · low angle (heroic) · high angle (diminished) · overhead / top-down · dutch (tension).

**Lens:** 24mm wide (dramatic, distorts) · 35mm (natural) · 50mm (neutral) · 85mm tele (compressed, portraits).

**Camera MOVE (for video):** static · slow push-in · pull-out · pan L/R · tilt up/down · orbit / arc ·
dolly · crane up/down · handheld · tracking follow · whip-pan.

---

## Model-specific consistency knobs

| Model | Lock identity with | Lock render with | Aspect |
|-------|--------------------|--------------------|--------|
| **Midjourney** | `--cref [sheet URL]` (character ref) + `--cw 100` | `--sref` (style ref), `--seed [n]` | `--ar 16:9` |
| **Gemini / "Nano Banana"** | attach the sheet as image input; say *"keep the same character, identical face & outfit"* | restate the style lock each turn | state in prompt |
| **Stable Diffusion** | IP-Adapter (FaceID) on the sheet, or train a LoRA | fixed `seed`, same checkpoint + sampler | width/height |
| **Seedream / Seedance** | `reference_image` role = the sheet | repeat style lock in prompt | `ratio` param |
| **DALL·E** | *"same character as before"* + re-use `gen_id` | restate style | size param |

> Universal fallback (works everywhere): paste the **Character Bible verbatim** + **same seed** +
> **attach the turnaround sheet** as a reference image. That trio carries consistency on any model.

---

## TL;DR for friends

1. Write the **Character Bible** once. Make the style **stylized, not photoreal**.
2. Generate a **turnaround sheet** (Stage 1). Save it — it's your reference image forever.
3. Storyboard with **one camera spec per panel** (Stage 2). Change only action + camera.
4. Animate **one beat per clip** (Stage 3): feed the sheet as reference, the panel as first frame,
   bible + camera move as the prompt.
5. Re-use the **same seed** and **attach the sheet** on every generation.
6. ⭐ For video that **doesn't look AI**: bolt on the cinematic-motion preset — camera *tracks with* the
   subject, background streaks past, cinematic motion blur, god-rays. Speed = background velocity, not post.
