---
name: continuity-director
description: "MUST use when the user hands a multi-scene storyboard, shot list, scene breakdown, or
             text script for an AI video and wants it consistent across shots — or says 'check
             continuity', 'fix continuity', 'continuity pass', 'lock lighting', 'color temperature',
             'environment continuity', 'set continuity', 'object/prop continuity', 'physics
             continuity', 'screen direction', 'make my scenes match', 'analyze my storyboard/script',
             'why do my scenes look different'. The CONTINUITY-ANALYSIS layer that runs BEFORE
             rendering: it locks the world, light, props & physics across every beat, lints the
             script for breaks (time/lighting jumps, color-temp drift, prop pop-in/out, 180°/screen-
             direction flips, scale/physics errors), and rewrites every beat with continuity tags.
             Character consistency is the separate STORYBOARD-PROMPT-KIT job — this skill owns
             everything else. Pairs with media-generation/storyboard-runner.sh (Seedance i2v) and the
             storyboard skill; it does NOT render — it emits a corrected .spec for storyboard-runner.sh (Seedance i2v)."
---

# Continuity Director — The World Must Match, Shot to Shot
*Character is one variable. The world, the light, and the laws of physics are the other five. A film that nails the face but jumps from afternoon to night reads as broken.*

> Invocable as `/continuity-director`. Runs the **continuity-analysis pass** on a storyboard/script
> **before** it goes to the renderer. Complements two existing pieces — it does not replace them:
> - **Character** identity lock → `media-generation/STORYBOARD-PROMPT-KIT.md` (the Character Bible).
> - **Rendering** (Seedance i2v, one clip/beat, stitch) → `media-generation/storyboard-runner.sh`.
>
> This skill is the missing middle: everything that is NOT the character's face/outfit.

## Activation

When this skill activates, output:

`🎞️ Continuity Director — lock world · light · color-temp · props · physics · screen-direction → lint for breaks → rewrite beats → hand to renderer.`

Then execute the protocol.

## Context Guard

| Context | Status |
|---------|--------|
| **Multi-scene storyboard / script handed in for an AI video** | ACTIVE — full protocol (run before render) |
| **"check / fix continuity", "lock lighting/color/set/props/physics"** | ACTIVE — full protocol |
| **"why do my scenes look different / not match"** | ACTIVE — lint + diagnose |
| **A single still / one shot** | DORMANT — no cross-shot continuity to lock; use `image-prompt` |
| **Character face/outfit drift only** | DORMANT — that is the Character Bible's job (`STORYBOARD-PROMPT-KIT`) |
| **The actual render / stitch step** | DORMANT — hand the corrected spec to `storyboard-runner.sh` |

## The Six Continuity Domains

A shot-to-shot break can come from any of these. The Character Bible covers #1; **this skill owns #2–#6.**

| # | Domain | What drifts if unlocked | Owner |
|---|--------|-------------------------|-------|
| 1 | **Character** | Face, hair, wardrobe, build, props-on-body | `STORYBOARD-PROMPT-KIT` (defer) |
| 2 | **Environment / Set** | A revisited location looks different; set dressing changes | THIS skill |
| 3 | **Lighting & Color Temperature** | Time-of-day jumps; key-light flips sides; warm→cool drift; grade changes | THIS skill |
| 4 | **Props / Objects** | A backpack/phone pops in and out; prop state changes between cuts | THIS skill |
| 5 | **Physics & Motion** | Scale, gravity, weather (rain/dust) discontinuity; impossible motion | THIS skill |
| 6 | **Camera / Spatial** | 180°-line violation (screen direction flips); eyeline mismatch; no match-on-action | THIS skill |
| 7 | **Narrative / Temporal** | beats render out of order; a later state (finish line, celebration) appears before it's earned; no momentum carried across cuts | THIS skill |

## Protocol

### Step 1: Parse & Extract
- [ ] Split the script into numbered **beats** (one beat = one shot/clip).
- [ ] For each beat extract: **location**, **time-of-day**, **light source(s)**, **characters present**, **props present + state**, **camera (shot/angle/lens/move)**, **action**, **weather/atmosphere**.
- [ ] Mark **revisited locations** (same place in 2+ beats) and **timeline links** (beats that are continuous in story-time vs. a deliberate time-skip).

### Step 2: Build the Continuity Bible
Author the lock tables. Each entry is written **once** and reused **verbatim** downstream.
- [ ] **Environment/Set Bible** — one locked paragraph per distinct location (architecture, materials, set dressing, palette). Revisits reuse the exact text.
- [ ] **Lighting & Color plan** — per beat: time-of-day phase, **Kelvin**, **key-light direction**, contrast/mood, palette. Derive from a single timeline arc (see Reference B). Hold one phase across a continuous sequence.
- [ ] **Prop ledger** — a row per object × beat: present / absent / state (e.g. `phone: in-hand, screen=app-feed`). Catches pop-in/out.
- [ ] **Physics rules** — scale anchors (character height vs set), gravity, weather persistence, motion plausibility.
- [ ] **Spatial map** — the action line per scene; screen direction for each moving subject; eyelines.

### Step 3: Continuity Lint (the Break Report)
- [ ] Scan **adjacent beats** and **same-location beats** for breaks. Tag each with severity:
  - 🔴 **Breaks story logic** — e.g. night sky in a sequence that is seconds after a daylight beat.
  - 🟡 **Inconsistent / distracting** — location drift on a revisit, prop pop-out, key-light flip.
  - 🟢 **Minor / polish** — slight palette or lens drift.
- [ ] Categories to check: time/lighting jump · color-temp drift · key-light direction flip · prop pop-in/out · wardrobe/state change · scale error · **180° / screen-direction flip** · eyeline mismatch · missing match-on-action · weather/physics discontinuity.

### Step 4: Resolve (surface forks, don't average)
- [ ] Each break → a concrete fix. Apply obvious ones.
- [ ] When a break could be an **intentional creative choice** (e.g. an emotional golden-hour ending after a daytime rescue), **surface it as a fork and ask/offer** — do NOT silently normalize (Rule 7).

### Step 5: Rewrite (inject the locks)
- [ ] **Prepend a CONTINUITY block** to the `@bible`: world look + global light/grade + recurring-location locks + persistent props.
- [ ] **Append per-beat continuity tags** to each beat: `LOCATION | TIME | LIGHT (dir) | TEMP (K) | PROPS | SCREEN-DIR`.
- [ ] Emit the corrected artifact: a `storyboard-runner` **`.spec`** (`@bible` + `@beats`) for Seedance i2v — or apply the same continuity locks + per-beat tags directly to the `storyboard` skill's storyboard **document**.

### Step 6: Handoff
- [ ] Hand the corrected `.spec` to `media-generation/storyboard-runner.sh` (Seedance i2v, character sheet as `--reference-image`). (Local ComfyUI/OpenMontage render via the `storyboard` skill is retired — that skill is now a doc-only director.)
- [ ] Report **what was locked** and **what was fixed** (the Break Report). Offer a re-lint after edits.

---

## Reference A — Color Temperature (Kelvin) cheat-sheet
Lock a Kelvin + palette per beat. Sudden warm↔cool jumps between continuous shots read as broken.

| Source / condition | Kelvin | Feel / palette |
|--------------------|--------|----------------|
| Candle / fire | ~1900K | deep amber |
| Tungsten / interior bulb | ~3200K | warm orange |
| **Golden hour (sunrise/sunset)** | **3000–4000K** | warm gold, long shadows |
| Early morning / late afternoon sun | ~4300K | soft warm |
| Midday sun | 5000–5600K | neutral white |
| Camera flash / nominal daylight | ~5500K | neutral |
| Overcast | ~6500K | flat cool-neutral |
| Open shade | 7000–8000K | cool blue |
| **Blue hour / twilight** | **8000–10000K** | deep blue |
| Moonlight (cinematic) | ~7000–8000K | cool silver-blue |
| Fluorescent office / ops center | 4000–5000K | greenish-neutral |
| **Screen / holographic glow** | **8000K+ cyan** | adds cyan rim — layer ON the base temp, don't replace it |

## Reference B — Time-of-Day / Lighting arc
A **continuous** sequence must hold one phase (or transition believably). Pick the phase, lock it across all beats in that timeline.

| Phase | Key-light angle | Shadows | Contrast | Palette |
|-------|-----------------|---------|----------|---------|
| Dawn / blue hour | very low, diffuse | long, soft | low | cool blue |
| Sunrise / golden AM | low, warm | long | medium | gold + cool sky |
| Midday | high, overhead | short, hard | high | neutral, saturated |
| Afternoon | descending | lengthening | medium | warm-neutral |
| Golden hour PM | low, warm, raking | long, warm | medium-high | amber/orange |
| Sunset | at horizon | very long | high | red-orange + magenta |
| Night | practicals/moon only | pooled | high local | blue base + warm pools |

## Reference C — Lighting continuity rules
1. **Key-light direction is constant** per location+time. If the sun is camera-left in beat 1, it is camera-left every beat in that scene. Flipping it = the 180° rule for light.
2. **Hold the contrast ratio / mood** within a scene (don't drift soft→hard).
3. **Practicals persist** — a glowing screen, a red alert lamp, a window shaft must appear consistently when in frame.
4. **Weather persists** — rain, dust, fog do not vanish between cuts of the same moment.
5. **Grade is global** — one color grade (LUT feel) across the whole film unless a deliberate look-shift is signposted.

## Reference D — Spatial / Camera continuity
1. **180-degree rule** — keep the camera on one side of the action line so **screen direction** stays consistent (a character facing/ moving screen-right keeps doing so across cuts).
2. **Screen direction of travel** — exit frame-left → enter next shot frame-right (continuous pursuit/travel).
3. **Eyeline match** — where a character looks in shot A must aim at what's revealed in shot B.
4. **Match-on-action** — motion started in one beat continues seamlessly into the next.
5. **Consistent lens language** — don't jump 24mm↔85mm at random; let lens serve the beat, not noise.

---

## Worked example — this is the ad that created this skill
The "ONE TAP CAN SAVE A LIFE" community-safety ad (8 beats, Seedance i2v). **Character held perfectly** (same Malay schoolgirl, tudung, pink backpack). But the non-character domains drifted:

| Break | Beats | Sev | Fix |
|-------|-------|-----|-----|
| **Night KL skyline** in a sequence that is seconds after a **daylight** chase | 3 → 4 | 🔴 | Lock beat 4 to the same afternoon timeline. Render the SOS as a **screen/holographic overlay or macro phone CU** (no day/night sky), OR an abstract blue holo-map — not a night cityscape. |
| **Location drift** — rescue happens in a different mural alley, not at her hiding spot | 6 → 7 | 🟡 | Reuse the beat-3/6 **construction-site/under-staircase** Environment lock verbatim for beat 7. |
| **Sunset** appears after a rescue that was minutes long | 7 → 8 | 🟡 fork | Could be an intentional emotional golden-hour ending. **Ask the user** — keep continuous afternoon, or signpost a short time-pass. |
| **Pink backpack** present in 1/2/3/6, ambiguous in 8 | all | 🟢 | Prop-ledger lock: backpack on her back through the rescue; on the ground beside her once safe. |
| **Color-temp** swings warm(1–3) → cool office(5) → night-cyan(4) → warm(8) with no plan | all | 🟡 | One arc: warm afternoon (~3800K) exteriors; ops-center interior ~4500K (justified, locked); hopeful warm finish. No night. |

**Continuity block this would prepend to `@bible`:**
```
CONTINUITY LOCK — one continuous late-afternoon, warm golden daylight (~3800K), sun low and
camera-left, long soft shadows; single warm cinematic grade. Recurring location "HIDE SITE":
an unfinished concrete construction site, raw grey pillars and a concrete staircase, dust in
shafts of low sun (beats 3, 6, 7 — identical). Persistent props: white tudung + baju kurung,
pink backpack on her back, smartphone with a pink case. Interior "OPS CENTER" (beat 5 only):
cool ~4500K screen-lit operations room, NO daylight, NO child present.
```
**Example rewritten beat (4):**
```
extreme close-up of the phone — large glowing red SOS button — trembling thumb presses it; a blue
holographic signal blooms from the screen across a translucent holo map overlay (no sky, no city
skyline)  || macro CU pushing in, holo overlay rising  || 5
   # LOCATION: HIDE SITE | TIME: late afternoon (continuous) | LIGHT: low warm, camera-left
   # TEMP: 3800K base + cyan holo glow | PROPS: phone(pink case, screen=SOS) | SCREEN-DIR: held
```

---

## Lv.2 — Timeline & Color-Temp Coherence Gate (automatic first pass)
Before the manual lint (Step 3), run this **deterministic gate** over the beat table and emit its flags at the **top of the Break Report** — so the expensive-to-miss classes are never skipped:
1. **Timeline / Kelvin jump** — within a run of story-continuous beats, any beat whose time-of-day or Kelvin band differs from its neighbours → 🔴 (the "night-in-afternoon" class — scene 4 of the origin ad).
2. **Revisited-location drift** — a location named in 2+ beats but described in different words → 🟡 (reuse verbatim).
3. **Prop pop-out** — an object present → absent → present across beats → 🟡.
4. **Key-light flip** — a stated light direction reversing within one location/time → 🟡.
Then refine with the full manual lint (Step 3). The gate is the safety net; the manual pass is the polish.

## Lv.3 — Narrative / Temporal continuity (the story-clock)
i2v renders every clip **independently** — it has no idea beat 4 comes before beat 5. Visual locks (light, set, props) do NOT fix this: the model will happily show the finish line in S4, "running to it" in S5, and a landing in S2. So lock the **story-clock** too:
1. **Story-position per beat.** Tag each beat with where it sits on the story timeline (race: at-the-line → early → mid → approaching → crossing → aftermath) and bake it into the beat text.
2. **Forbid later-states-early (and regression).** "no finish line in view yet" (early), "they have NOT crossed yet" (approach), "past the line" (aftermath). Never let a later state appear early, nor an earlier state return.
3. **Action bridges.** End-state of beat N = start-state of beat N+1 — position, momentum, emotion carry across the cut ("still airborne and ahead" → the next beat opens airborne and ahead).
4. **Each beat advances** — establish → escalate → resolve; none repeats or stalls.
Run this in both the Coherence Gate and the manual Lint. **Seedance prompt-craft** (studied from the `awesome-seedance-prompts` repo, folded in): optional **timecoded shot lists** for a multi-shot single generation (`[00:00–00:05] Shot 1…`); **physical detail over abstraction** (describe what the camera sees, not "sad"); **"instead of"** to pin a look ("real feathers, instead of cartoon"); **contrast** (cool/warm light, slow-mo/real-speed) for shot readability.

## Mandatory Rules
1. **Continuity ≠ character.** The Character Bible (`STORYBOARD-PROMPT-KIT`) owns the face/outfit. This skill locks the other five domains — never duplicate the character work.
2. **Lint before lock.** Always produce the Break Report (Step 3). Never silently "fix" — show what was wrong.
3. **Surface creative forks, don't average** (Rule 7). A time-of-day or location change that might be intentional → ask/offer, don't normalize it away.
4. **Repeat verbatim.** Same location/prop reused word-for-word across beats. Re-describing loosely = drift.
5. **Respect the timeline.** Flag every time-of-day / Kelvin jump in a continuous sequence (the headline bug: night inside an afternoon).
6. **Hand off, don't render.** Emit the corrected `.spec`; defer rendering to `storyboard-runner.sh` (Seedance i2v).
7. **Bake continuity into the POSITIVE prompt.** Seedance (BytePlus ModelArk) has **no negative-prompt field** — locks must live in the prompt text, not a neg list.
8. **Fail loud** (Rule 12). If the script lacks the info to lock a variable (no stated time-of-day, ambiguous location), say so and ask — don't fabricate a value silently.
9. **Consult the lessons cache.** Read `media-generation/RENDER-LESSONS.md` and fold its Pre-flight checklist into the lint + locks — known render failures get pre-corrected from text, not re-discovered by eye. The `render-critic` skill keeps that file current.
10. **Lock the story-clock** (Lv.3). i2v has no cross-clip memory — tag every beat's story-position, forbid later-states-early, and bridge momentum across cuts. Visual continuity without narrative continuity still renders out of order (the jungle-bird-race lesson).

## Edge Cases
| Situation | Behavior |
|-----------|----------|
| Script gives no time-of-day | Infer a single phase from cues, state the assumption, ask to confirm before locking. |
| A location legitimately changes (travel montage) | Not a break — lock each location separately; ensure the transition reads as deliberate. |
| Deliberate flashback / dream (different grade) | Allowed — signpost it (grade shift, vignette) and lock the flashback's own look. |
| Multi-character continuity (bullies, responders) | One reference sheet locks ONE character; extras are prompt-described. Don't feed the protagonist ref to a protagonist-absent beat (it bleeds her in — see beat 5 "NO child present"). |
| User only wants the report, not a rewrite | Stop after Step 3 (Break Report) — deliver findings, skip the rewrite. |

## Level History
- **Lv.3** — Narrative/Temporal continuity, the 7th domain (2026-06-29): the **story-clock**. Each beat gets a story-position; later states can't appear early; action bridges carry momentum across cuts; beats must advance (establish→escalate→resolve). Folds in Seedance prompt-craft from the `awesome-seedance-prompts` repo (timecodes, physical-detail-over-abstraction, "instead of" disambiguation, contrast-for-clarity). (Origin: user caught the jungle-bird-race render playing out of order — finish line in S4, "running to finish" in S5, birds landing in S2 — because i2v renders clips independently with no story-clock: "I don't feel scene continuity… not continuous.")
- **Lv.2** — Coherence Gate (2026-06-29): a deterministic automatic first-pass that flags the high-cost break classes (timeline/Kelvin jump, revisited-location drift, prop pop-out, key-light flip) at the top of the Break Report before the manual lint. (Origin: user "increase level all video generation skills".)
- **Lv.1** — Base: six-domain continuity model (character defer + environment, lighting/color-temp, props, physics, camera/spatial). Parse→Bible→Lint(Break Report w/ 🔴🟡🟢)→Resolve(surface forks)→Rewrite(continuity block + per-beat tags)→Handoff to `storyboard-runner`. Kelvin + time-of-day + spatial reference tables baked in. (Origin: the "ONE TAP CAN SAVE A LIFE" ad, 2026-06-29 — character was consistent but the world drifted: a night skyline rendered inside an afternoon timeline, the rescue location wandered, color temperature swung with no plan. User: "study this and make it a great skill that fixes all the variables every time I hand over a storyboard.")
