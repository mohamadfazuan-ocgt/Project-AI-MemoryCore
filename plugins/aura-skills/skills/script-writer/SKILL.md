---
name: script-writer
description: "MUST use when the user wants to turn an idea/brief/product into a shot-by-shot SCRIPT
             for an AI video — director's notes, not a screenplay. Triggers on 'write a script',
             'script writing', 'scene breakdown', 'director's notes', 'shot list from this idea',
             'turn this into scenes', 'script for my video/ad', 'write 8 scenes', 'Seedance prompt',
             'write the scenes', 'storyboard the script'. This is STAGE 0 of the video pipeline — it
             WRITES the creative (logline → structure → 8–15 scenes, each as a Seedance 2.0 prompt
             line). It does NOT lock cross-shot continuity (that's continuity-director), does NOT
             lock the character's exact face/outfit (that's the Character Bible / STORYBOARD-PROMPT-
             KIT), and does NOT render (that's storyboard-runner.sh). Hand its output downstream."
---

# Script Writer — Director's Notes for AI Video
*Don't write a screenplay. Write director's notes. One idea per shot, present tense, built to render.*

> Invocable as `/script-writer`. **Stage 0** of the video pipeline:
> `script-writer` (write scenes) → `continuity-director` (lock world/light/props/physics) →
> Character Bible / `STORYBOARD-PROMPT-KIT` (lock identity) → `storyboard-runner.sh` (render i2v + stitch).
> This skill produces the *words*; the others lock and render them.

## Activation

When this skill activates, output:

`✍️ Script Writer — logline → structure → 8–15 director's-note scenes, each a Seedance 2.0 prompt line → hand to continuity-director.`

Then execute the protocol.

## Context Guard

| Context | Status |
|---------|--------|
| **Idea / brief / product → scene-by-scene script for a video** | ACTIVE — full protocol |
| **"director's notes", "scene breakdown", "write the scenes / shot list"** | ACTIVE — full protocol |
| **Rewrite / tighten an existing scene script** | ACTIVE — punch up + reformat to the formula |
| **Lock continuity across the scenes** (set/light/props) | DORMANT — hand to `continuity-director` |
| **Lock the character's exact look** | DORMANT — that's the Character Bible (`STORYBOARD-PROMPT-KIT`) |
| **Render / stitch the video** | DORMANT — hand the breakdown to `storyboard-runner.sh` |
| **A full narrative screenplay (dialogue-heavy, INT/EXT slugs)** | DORMANT — this writes director's notes, not screenplays |

## The core method (the user's directive, verbatim)

**Write director's notes, NOT a screenplay.** For each of **8–15 scenes**, capture five fields:

| Field | What to capture |
|-------|-----------------|
| **Subject** | Who is in the scene, what they look like, what they are wearing |
| **Action** | What happens — **present tense, ONE primary movement per shot** |
| **Camera** | Framing (wide / medium / close-up) + movement (dolly-in, pan, orbit, handheld) |
| **Mood & Style** | Lighting tone, colour palette, genre feel |
| **Audio** | Dialogue, ambient sound, music tone |

Then compose each scene into one **Seedance 2.0 prompt line** using the formula:

```
[Subject] + [Action] + [Environment] + [Camera Movement] + [Mood/Style] + [Audio Direction]
```

**Reference example (the gold standard for tone + density):**
> "A detective in a worn leather coat steps into a rain-soaked alley at midnight, camera tracking
> low at ankle height, high contrast noir lighting, tension building in the underscore, distant
> sirens fading into silence."

## Protocol

### Step 1: Parse the brief
- [ ] **Subject/product**, **goal/message** (what the viewer should feel/do), **genre/tone**, **platform + aspect** (default 16:9), **target length** → **scene count** (~5s/shot → 8–15 scenes ≈ 40–75s).
- [ ] If goal, tone, or length is missing → **ask before inventing** (Rule: fail loud).

### Step 2: Logline + structure (the spine)
- [ ] One-sentence **logline**.
- [ ] Pick a **structure template** (see Reference B) and lay the beats: e.g. ad = **Hook → Problem → Tension → Solution → Payoff → Tagline/CTA**.

### Step 3: Break into 8–15 scenes
- [ ] **One primary action per scene** (this becomes one clip downstream — never cram two beats into a shot).
- [ ] Present tense, active verbs, **show don't tell**, escalate scene to scene.

### Step 4: Fill the five director's-note fields per scene
- [ ] Subject · Action · Camera · Mood&Style · Audio — concise, concrete, renderable.
- [ ] **Subject = role + key visual only.** The exact face/wardrobe lock is the Character Bible's job — name the look, don't over-specify identity (avoid drift vs. the Bible).

### Step 5: Compose the Seedance prompt line
- [ ] Assemble each scene via `[Subject] + [Action] + [Environment] + [Camera Movement] + [Mood/Style] + [Audio Direction]` — one flowing sentence, like the detective example.

### Step 6: Handoff
- [ ] Emit the full breakdown (table of scenes + the prompt lines).
- [ ] **Map to the next stage**: the scene lines → `continuity-director` (lock set/light/props/physics, lint) → then `storyboard-runner` `@beats` (`motion || camera || seconds`).
- [ ] Offer: adjust tone, re-cut scene count, punch up a weak scene, or push the whole thing through continuity + render.

---

## Reference A — The six formula slots
| Slot | Pulls from | Note |
|------|-----------|------|
| **Subject** | director-note *Subject* | role + key visual; identity locked later by the Bible |
| **Action** | director-note *Action* | one primary movement, present tense |
| **Environment** | *Mood&Style* + scene location | where it happens; `continuity-director` locks the exact set |
| **Camera Movement** | director-note *Camera* | framing + move (see kit's camera cheat-sheet) |
| **Mood / Style** | director-note *Mood&Style* | lighting tone, palette, genre |
| **Audio Direction** | director-note *Audio* | dialogue / ambient / music — **real on Seedance 2.0** |

## Reference B — Structure templates (pick one, lay the beats)
| Template | Beat spine | Best for |
|----------|-----------|----------|
| **Ad / Commercial** | Hook → Problem → Tension → Solution → Payoff → Tagline/CTA | products, apps, PSAs |
| **Narrative short** | Setup → Inciting incident → Rising action → Climax → Resolution | story films |
| **Explainer** | Question/pain → Concept → How it works → Benefit → CTA | SaaS, how-it-works |
| **Montage / music video** | Theme statement → escalating image beats → crescendo → button | brand mood, hype |
| **Before/After** | Old painful way → turn → new way → proof → CTA | transformation pitches |

## Reference C — Craft rules (what "good director's notes" means)
1. **One idea per shot.** If a scene has two actions, split it into two scenes (one clip each downstream).
2. **Present tense, active verbs.** "She presses the button," not "the button would be pressed."
3. **Show, don't tell.** Externalize emotion as visible action ("eyes well up", not "she is sad").
4. **Specific > generic.** "worn leather coat, rain-soaked alley, midnight" beats "a man outside".
5. **Motivation per shot** — every cut earns its place in the spine; cut anything that doesn't escalate.
6. **Density like the example** — one sentence can carry subject + action + environment + camera + mood + audio.
7. **Camera serves the beat** — push-in for tension, pull-out for reveal/relief, handheld for chaos, static for dread. (Vocab: `STORYBOARD-PROMPT-KIT.md` camera cheat-sheet — reuse, don't reinvent.)

## Reference D — Audio direction (don't skip it)
Seedance 2.0 (`dreamina-seedance-2-0-260128`) **generates audio by default** (`generate_audio:true`). So the Audio field is *functional*, not decoration. Write three layers when relevant:
- **Dialogue** — short, only if it must be heard (lip-sync is imperfect; prefer VO/ambient for ads).
- **Ambient** — the world's sound (rain, sirens, schoolyard chatter, room tone).
- **Music tone** — underscore mood ("tension building", "hopeful swell"), not a specific track.

---

## Worked example — mini breakdown (3 of 8 scenes, ad spine)
**Logline:** A schoolgirl in danger taps one button and a community brings her home.

| # | Subject | Action | Camera | Mood & Style | Audio |
|---|---------|--------|--------|--------------|-------|
| 1 | Malay schoolgirl, tudung, pink backpack | walks home scrolling a safety app, smiling | wide, slow tracking dolly | warm golden afternoon, hopeful | schoolyard ambient, light music |
| 2 | same girl + three teen bullies | bullies close in and ring her | medium, slow push-in | same afternoon, tension rising | ambient drops out, low drone |
| 4 | the girl's phone | her thumb presses a glowing red SOS | extreme close-up, macro push-in | warm base + cyan UI glow | a sharp UI chime, underscore stings |

**Composed Seedance lines:**
```
S1: A Malay schoolgirl in a white tudung with a pink backpack walks home scrolling a community-safety
    app and smiling, golden late-afternoon schoolyard, slow tracking dolly following her, warm hopeful
    tone, schoolyard ambience under light music.
S2: The same schoolgirl is ringed by three teenage bullies who close in and block her path, the same
    warm alley turning tense, slow creeping push-in to a medium shot, dread building, ambience falling
    away under a low drone.
S4: Extreme macro close-up of her phone as a trembling thumb presses a large glowing red SOS button,
    warm light with a cyan interface glow, fast push-in on the screen, a sharp alert chime cutting the
    underscore.
```
Hand these to `continuity-director` to lock the set/light/props (e.g. keep S4 in the same afternoon, no night skyline) → then `storyboard-runner` renders them.

## Lv.2 — Emit a render-ready `.spec` (the handoff artifact)
When asked to "make the spec" / "render-ready" (or when pushing to the renderer), convert the breakdown straight into a `storyboard-runner` spec — no manual retyping:
- **`@bible`** ← the global look + subject in plain words, ONE paragraph reused identically every line (medium · lighting · palette · the recurring subject). Every beat inherits it.
- **`@beats`** ← one line per scene: `<Action as continuous present-tense motion>  ||  <Camera framing + movement>  ||  <seconds>`.
- **Durations** ← default `5`; bump to `6–8` only for a beat needing a longer move (reveal/crane). Sum = runtime.
- Then route the spec through `continuity-director` (lock set/light/props) → `storyboard-runner.sh`.
```
@beats
the same schoolgirl is ringed by three teenage bullies who close in and block her path  || slow creeping push-in to a medium shot  || 5
```

## Mandatory Rules
1. **Director's notes, not a screenplay.** Five fields + a formula line per scene — no INT/EXT slug-line scripts, no dialogue-heavy pages.
2. **One primary action per shot** — maps 1:1 to one clip downstream. Two actions → two scenes.
3. **Present tense, active verbs**, show-don't-tell, specific.
4. **Subject = role + key visual.** Defer the exact face/outfit lock to the Character Bible; don't invent identity details that will drift.
5. **Always write Audio** — Seedance 2.0 generates sound (`generate_audio:true`); silent-by-omission is a mistake unless silence is the intent.
6. **Every scene ends as a Seedance formula line** (`[Subject]+[Action]+[Environment]+[Camera Movement]+[Mood/Style]+[Audio Direction]`).
7. **Scene count 8–15**, mapped to length (~5s/shot). Flag if the brief implies fewer/more.
8. **Hand off, don't render** — output the breakdown; route to `continuity-director` then `storyboard-runner`. Never call the paid render from here.
9. **Fail loud** — missing goal/tone/length → ask, don't fabricate.
10. **Consult the lessons cache.** Before emitting the spec, read `media-generation/RENDER-LESSONS.md` and apply its Pre-flight checklist (cheap text) — the script is pre-corrected for known Seedance behaviour with no vision pass needed.

## Edge Cases
| Situation | Behavior |
|-----------|----------|
| User gives only a one-line idea | Draft logline + spine, propose a scene count, then write — but confirm tone/length first. |
| Brief implies > 15 scenes | Write the spine, flag that >15 shots = longer + pricier render; propose a tighter cut. |
| Dialogue-heavy concept | Warn that AI-video lip-sync is weak; recommend VO/ambient-led storytelling, keep on-screen dialogue minimal. |
| User wants a literal screenplay | Out of scope — offer the director's-notes form instead, or note this skill targets renderable shot scripts. |
| Existing script handed in | Skip to Step 4–5: reformat each scene into the five fields + the Seedance line, punch up weak beats. |

## Level History
- **Lv.2** — Spec Emission (2026-06-29): converts the breakdown directly into a render-ready `storyboard-runner` `.spec` (`@bible` + `@beats` as `motion || camera || seconds`, default 5s), then routes through `continuity-director` → `storyboard-runner.sh`. Closes the gap from script to render. (Origin: user "increase level all video generation skills".)
- **Lv.1** — Base: idea/brief → logline → structure template → **8–15 director's-note scenes** (Subject/Action/Camera/Mood&Style/Audio) → one **Seedance 2.0 formula line** per scene → handoff to `continuity-director` → `storyboard-runner`. Craft rules (one-action-per-shot, present-tense, show-don't-tell), structure templates, and audio-is-real guidance baked in. (Origin: user "aura learn script writing skills also", 2026-06-29 — supplied the director's-notes method + the `[Subject]+[Action]+[Environment]+[Camera]+[Mood/Style]+[Audio]` formula + the detective-noir gold-standard example. Completes the upstream of the video pipeline, ahead of the same day's continuity-director.)
