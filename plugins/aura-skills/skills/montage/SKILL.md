---
name: montage
description: "MUST use when the user asks to compose, edit, or assemble a finished video from
             parts — a montage, explainer, or multi-scene clip with titles, captions, narration,
             transitions, or stitched shots. Triggers on 'make a montage', 'compose a video',
             'explainer video', 'add titles/captions', 'stitch these clips', 'video from script',
             or references to OpenMontage on dgx-spark. Drives the deployed OpenMontage container
             (Remotion + Piper + ffmpeg, CPU) and routes the raw-clip-gen step to the ComfyUI /
             Wan2.2 engine via the comfyui_video bridge. For a SINGLE raw clip with no
             composition, use video-generation instead."
---

# Montage — OpenMontage Composition Layer
*Script in, finished video out. Compose on CPU; borrow the GPU only for the gen step.*

## Activation

When this skill activates, output:

`🎞️ Montage — composing on OpenMontage (Remotion + ffmpeg), gen via the ComfyUI bridge.`

Then execute the protocol below.

## Context Guard

| Context | Status |
|---------|--------|
| **Compose a multi-part video** (titles, captions, narration, transitions, stitched shots) | ACTIVE — full protocol |
| **Turn a script / storyboard into a finished video** | ACTIVE — full protocol |
| **Single raw AI clip, no composition** | DORMANT — use `video-generation` |
| **Still image / Midjourney prompt** | DORMANT — use `image-prompt` |
| **Non-video request** | DORMANT |

## What this is (and isn't)

OpenMontage **composes** — it does not generate pixels. It arranges generated clips + stock +
Remotion animation (titles, lower-thirds, data-viz, captions) + narration into a finished video.
The raw-clip **generation** step is delegated to the ComfyUI / Wan2.2 engine through the
`comfyui_video` bridge. **Stack, don't swap** — ComfyUI stays the only GPU generator; this layer
is CPU-only. Full design: `dgx-ai-ocgt/docs/14`.

## Engine Reference (deployed)

- **Container**: `aiplat-openmontage-openmontage-1` on `dgx-spark` (net `aiplat`, CPU-only, **no GPU
  reservation**). No HTTP server — **exec-driven**.
- **Gen bridge**: `tools/video/comfyui_video.py` (`provider=comfyui`) → drives `http://comfyui:8188`
  (Wan2.2-ti2v-5b), submit/poll/download. Workflow template at `tools/video/workflows/`.
- **Compose**: Remotion (`remotion-composer/`, React→mp4, Chromium-headless baked for arm64) +
  ffmpeg. **TTS**: Piper (local, CPU). All air-gapped.
- **Drive pattern**: `sudo docker exec -i aiplat-openmontage-openmontage-1 python3 <<'PY' … PY`.
- **Constraints**: docker needs interactive sudo (no headless); outputs persist in the
  `openmontage-work` volume (`/app/projects`); source media drops into `/srv/openmontage/input`.

## Protocol

### Step 1: Parse Request
- [ ] Subject, narrative structure (scenes/beats), target length
- [ ] Which parts are **gen** (AI clips) vs **Remotion** (titles/cards/data-viz/captions)?
- [ ] Narration? (Piper TTS) · Captions? · Music/SFX? (user-supplied track only)
- [ ] **Style** — photoreal or anime → set the gen profile (delegated to `video-generation`)

### Step 2: Check Engines
- [ ] `sudo docker ps | grep openmontage` → up + healthy
- [ ] Reach ComfyUI from the container: `GET http://comfyui:8188/system_stats` → 200
- [ ] If either down → surface it, do not fabricate

### Step 3: Plan Composition
- [ ] Shot list: per shot = source (gen / stock / Remotion), duration, on-screen text
- [ ] Fix character + setting descriptions, reuse across gen beats (continuity)

### Step 4: Generate clips (per beat)
- [ ] Via the `comfyui_video` bridge — **apply the `video-generation` style profile** (photoreal
      or anime) for prompt + negative + settings; `length ≤ 121` (~5s) per clip
- [ ] >5s total → beat-split into ≤5s shots; ComfyUI serializes the queue (no parallel OOM)

### Step 5: Narration + assets
- [ ] Piper TTS for narration if requested · gather stock/Remotion assets

### Step 6: Compose (Remotion)
- [ ] Build the composition (titles, lower-thirds, captions, transitions, data-viz)
- [ ] `npx remotion render` → mp4

### Step 7: Stitch + mux (ffmpeg)
- [ ] Concat beats in order; transitions per content (hard cuts suit action)
- [ ] Mux audio only if narration/track exists; ffmpeg runs in the container (sudo exec)

### Step 8: Retrieve + Confirm
- [ ] Pull the final mp4 (`docker cp`); verify valid (`ffprobe`) before claiming success
- [ ] Report path + dims/frames/duration; offer: tune style, re-cut, more beats, add sound

## Mandatory Rules

1. **Never fake a render** — only report a file that exists and probes valid. Engine down / poll
   timeout → say so.
2. **Drive via heredoc, never inline** — `docker exec -i … python3 <<'PY'`; long `-c "…"` strings
   get newline-wrapped by the terminal → `SyntaxError` / `unterminated string literal`.
3. **CUDA poison → restart, never retry** — `CUDA error: operation not permitted` at KSampler =
   poisoned context → `sudo docker restart comfyui`, resubmit. (`/system_stats` healthcheck can
   read green while compute is broken.)
4. **5s ceiling per clip** — `length ≤ 121`; longer → beat-split + ffmpeg concat. Surface the split.
5. **Apply the gen style profile** (`video-generation`: photoreal vs anime); don't ship raw default
   prompts — that's what "not real enough" looks like.
6. **Sudo is manual** — print `sudo docker exec` / build commands; never assume headless sudo.
7. **No-auth engine** — `:8188` LAN-only; never publish results externally without asking.
8. **AGPL** — OpenMontage is AGPL-3.0; internal/LAN use only, no public network service without
   source disclosure.

## Edge Cases

| Situation | Behavior |
|-----------|----------|
| **ComfyUI `operation not permitted`** | Poisoned context → `sudo docker restart comfyui`, resubmit |
| **`node_errors` on submit** | Graph mismatch for the ComfyUI version → fix `tools/video/workflows/*.json` (probe `/object_info/{Class}`) |
| **Output not real enough** | Wrong/weak profile → switch photoreal↔anime, raise steps, strengthen negative; LoRA if it plateaus (M6) |
| **>5s requested** | Beat-split; never one-shot |
| **i2v (animate a still)** | Bridge is t2v today; i2v needs the `wan22-ti2v-5b.i2v.json` template + `/upload/image` (TODO) |

## Level History

- **Lv.1** — Base: drive the deployed OpenMontage container to compose finished videos —
  plan → gen clips via the `comfyui_video` bridge (profile-aware) → Remotion compose →
  Piper narration → ffmpeg stitch → verify. Complements `video-generation` (raw clips).
  (Origin: 2026-06-22, OpenMontage PATH-B integration on dgx-spark.)
