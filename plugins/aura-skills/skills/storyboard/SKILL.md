---
name: storyboard
description: "MUST use when the user asks to direct or build a storyboard — a script + scene/shot
             breakdown + per-shot keyframe stills — for handoff to an external video API (Seedance
             2.0 / Higgsfield). Triggers on 'storyboard', 'shot list', 'scene breakdown',
             'keyframes', 'direct a video', 'script for a video', 'make a video', 'generate video',
             'create anime', 'text to video', 'animate this', 'make a montage', 'compose a video',
             'explainer video', or references to OpenMontage on dgx-spark. The director
             (this skill, client-side) writes the script + shots + prompts and emits
             storyboard.json; the deployed OpenMontage container renders every still via the
             ComfyUI / Flux bridge (comfyui_image) and builds a rough animatic preview. Local
             video generation is retired — final motion is external. For a SINGLE still, use
             image-prompt; the legacy local-video skill video-generation is DEPRECATED."
---

# Storyboard Director — OpenMontage Storyboard Layer
*Prompt in, directed storyboard out (script + shots + keyframe stills + animatic). Hand off to an external video API for the motion.*

> Invocable as `/storyboard`. The skill **directs storyboards**, not finished
> videos — local video gen is retired (see `dgx-ai-ocgt/docs/14`, pivot 2026-06-23).

## Activation

When this skill activates, output:

`🎬 Storyboard Director — script → shots → ComfyUI/Flux stills → package + animatic → (external) Seedance/Higgsfield.`

Then execute the protocol below.

## Context Guard

| Context | Status |
|---------|--------|
| **Direct a storyboard** (script, shots, keyframe stills, for external video) | ACTIVE — full protocol |
| **Turn a prompt/idea into a shot-by-shot board** | ACTIVE — full protocol |
| **Rough animatic / pacing preview from stills** | ACTIVE — `storyboard_gen.py --animatic` |
| **Single still image / Midjourney prompt** | DORMANT — use `image-prompt` |
| **Raw local AI *video* clip** | DORMANT — `video-generation` is DEPRECATED (local video retired) |
| **Non-storyboard request** | DORMANT |

## What this is (and isn't)

The **director runs client-side** (this skill / the LLM here) and writes the *creative*: a script,
a scene/shot breakdown, and per-shot **start + end keyframe prompts**. It emits a
**`storyboard.json`** (pure data). The **DGX Spark box stays Claude-free / air-gapped**: it only
**renders** the stills (ComfyUI / Flux via the `comfyui_image` bridge) and builds an **animatic**
(ffmpeg + Piper). The **final video is external** — the package is handed to **Seedance 2.0** or
**Higgsfield** (first/last-frame i2v). **Stack, don't swap**; ComfyUI is the only GPU generator.
Full design: `dgx-ai-ocgt/docs/14`.

> Anyone using this runs their **own** client-side director against the shared box. Do **not** try
> to generate video on the box — that path is retired by design.

## Engine Reference (deployed)

- **Container**: `aiplat-openmontage-openmontage-1` on `dgx-spark` (net `aiplat`, CPU-only, **no GPU
  reservation**). No HTTP server — **exec-driven**.
- **Image bridge**: `tools/video/comfyui_image.py` (`provider=comfyui`) → `http://comfyui:8188`
  (Flux.1-schnell fp8), submit/poll/download. Workflow: `tools/video/workflows/flux1-schnell.txt2img.json`.
- **Renderer/packager**: `/app/storyboard_gen.py --manifest <path> [--animatic] [--narrate]`.
- **Animatic**: ffmpeg Ken-Burns slideshow + Piper TTS (best-effort, silent fallback). All air-gapped.
- **Constraints**: docker needs interactive **sudo** (no headless); packages persist in
  `openmontage-work` (`/app/projects/<slug>/`).

## Style profiles (the look — the director owns these)

Build each shot's prompt as `lead + <character block> + <shot action> + tail`; pass `negative` +
`settings` in the manifest's `style`. Flux schnell defaults: `steps 4, cfg 1, euler, simple`.

- **photoreal** — lead `photorealistic cinematic film still,` · tail `, shot on 35mm, shallow depth
  of field, natural volumetric lighting, physically-based materials, detailed texture, color-graded
  like a feature film` · negative `anime, cartoon, illustration, 3D render, CGI, plastic, waxy skin,
  oversaturated, lowres, blurry, distorted anatomy, extra limbs, bad hands, watermark, text`.
- **anime** — lead `pure 2D hand-drawn Japanese anime, cel-shaded, flat 2D cel art,` · tail `,
  vibrant anime color grading, dramatic anime lighting` · negative `3D, CGI, realistic humans, live
  action, plastic, distorted anatomy, extra limbs, bad hands, blurry, watermark, text, photorealistic,
  western cartoon, chibi, low quality`.
- **cinematic** (ufotable-ish dark fantasy) — lead `cinematic dark-fantasy anime, high-budget
  ufotable look, 2D cel-shaded characters with 3D digital FX,` · tail `, volumetric glow and bloom,
  neon gold/violet accents, rain and haze, high-contrast chiaroscuro, dark moody grade` · negative
  `flat lighting, washed out, static, chibi, cute, daytime, pastel, 3DCG characters, plastic, live
  action, distorted anatomy, blurry, watermark, text, low quality`.

(Reuse the learned `anime-video-style` memory language; it now feeds **stills**, not local video.)

## Protocol

### Step 1: Parse Request
- [ ] Subject/genre, target shot count (or length → ~`length/duration_per_shot`), style, aspect (default **16:9**)
- [ ] Any fixed characters/setting → write a **verbatim character block** (reused every shot)

### Step 2: Direct (write the script)
- [ ] Logline + a short script (scene/beat structure)
- [ ] Scene/shot breakdown: per shot = `id`, `scene`, `script` line, `camera`, `motion`, `duration_s`

### Step 3: Author per-shot prompts
- [ ] For each shot: `start.prompt` (frame 1) + `end.prompt` (last frame) = `lead + character block +
      shot action + tail`
- [ ] **Lock one seed per shot** (start + end share it) → composition stability
- [ ] Reuse the **same character block** string across all shots → cross-shot consistency

### Step 4: Assemble `storyboard.json`
- [ ] Provider-agnostic schema (docs/14 §14.3): `resolution` (1280×720), `fps`, `style`
      (lead/tail/negative/settings), `characters[]`, `shots[]` (start/end prompts + seed + duration)

### Step 5: Check Engines
- [ ] `sudo docker ps | grep openmontage` → up · `GET http://comfyui:8188/system_stats` → 200
- [ ] Either down → surface it, do not fabricate

### Step 6: Drive the box (print sudo; never execute it)
- [ ] Write the manifest into `/app/projects/<slug>/storyboard.json` via **heredoc** (`docker exec -i … sh -c 'cat > …'`)
- [ ] `python3 /app/storyboard_gen.py --manifest /app/projects/<slug>/storyboard.json --animatic`

### Step 7: Retrieve + verify
- [ ] `docker cp` the package out; confirm every shot has `images/<id>_start.png` + `_end.png`
- [ ] `animatic.mp4` probes valid (`ffprobe`); manifest copied with image paths + actual seeds

### Step 8: Confirm + handoff
- [ ] Report package path, shot count, stills count; show/offer the animatic
- [ ] **Handoff note**: feed `storyboard.json` + `images/` to **Seedance 2.0 / Higgsfield** (per-shot
      start+end frame i2v). Offer: tune style, re-cut shots, add/adjust shots, regenerate a still

## Mandatory Rules

1. **Never fake a render** — only report stills/animatic that exist and probe valid. Engine down /
   timeout → say so (Rule 12).
2. **Director is client-side; box is Claude-free** — write the creative here, send only data
   (`storyboard.json`). Never propose serving Claude on the Spark.
3. **Final video is external** — never generate video on the box; the package goes to Seedance/Higgsfield.
4. **Drive via heredoc, never inline** — `docker exec -i … <<'PY/JSON'`; long `-c "…"` gets
   newline-wrapped → `SyntaxError`.
5. **CUDA poison → restart, never retry** — a CUDA error at KSampler = poisoned context →
   `sudo sh -c "docker restart $(docker ps -q --filter publish=8188)"`, then resubmit.
6. **Consistency is your job** — lock one seed per shot + reuse the verbatim character block. Two
   independent Flux renders drift otherwise.
7. **Apply a style profile** — never ship raw default prompts (that is what "not good enough" looks like).
8. **Sudo is manual** — print `sudo docker exec` / `docker cp` / build commands; never assume headless sudo.
9. **No-auth engine** — `:8188` LAN-only; never publish results externally without asking.
10. **AGPL** — OpenMontage is AGPL-3.0; internal/LAN use only.

## Edge Cases

| Situation | Behavior |
|-----------|----------|
| **ComfyUI CUDA error / `operation not permitted`** | Poisoned context → restart ComfyUI container, resubmit. Never retry in place. |
| **`node_errors` on submit** | Flux graph mismatch for the ComfyUI version → fix `flux1-schnell.txt2img.json` (probe `/object_info/{Class}`; e.g. `EmptySD3LatentImage`, `CheckpointLoaderSimple` indices) |
| **Still not good enough** | Strengthen the style profile (lead/tail/negative); raise steps; consider a LoRA if it plateaus |
| **Start/end frames look like different characters** | Lock seed per shot + reuse the character block verbatim; upgrade = img2img end-frame off the start (TODO, docs/14 §14.4) |
| **Piper unavailable (arm64)** | Animatic degrades to a **silent** slideshow — logged, not failed |
| **User wants the finished video now** | Out of scope on-box — produce the package, hand to Seedance/Higgsfield |

## Level History

- **Lv.2** — Pivot (2026-06-23): retired local video composition; became the **storyboard director**.
  Directs script + shots + start/end keyframe prompts → `storyboard.json` → box renders stills via
  the new `comfyui_image` (Flux) bridge + `storyboard_gen.py` → package + ffmpeg animatic → handoff
  to external video API (Seedance 2.0 / Higgsfield). Style profiles moved here (director owns the look).
- **Lv.1** — Base (2026-06-22): drove OpenMontage to compose finished videos via the Wan2.2
  `comfyui_video` bridge (Remotion compose + Piper + ffmpeg stitch). Now legacy — see docs/14 §14.10.
