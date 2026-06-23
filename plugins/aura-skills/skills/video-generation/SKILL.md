---
name: video-generation
description: "MUST use when user asks to create or generate a video, anime video, text-to-video,
             image-to-video, or AI video clip, when user says 'make a video', 'generate video',
             'create anime', 'text to video', 'animate this', 'video from prompt', or references
             the ComfyUI / Wan video engine on dgx-spark. Runs the deployed Wan2.2 pipeline over
             the ComfyUI HTTP API (submit, poll, download) and handles >5s clips via beat-splitting."
---

# Video Generation — Wan2.2 / ComfyUI Pipeline
*Prompt in, anime out. Drive the GPU engine over HTTP — no GUI.*

## Activation

When this skill activates, output:

`🎬 Video Generation — driving Wan2.2 on the ComfyUI engine.`

Then execute the protocol below.

## Context Guard

| Context | Status |
|---------|--------|
| **User asks to create/generate a video or anime clip** | ACTIVE — full protocol |
| **User gives a scene/storyboard to animate** | ACTIVE — full protocol |
| **User asks to tune/restyle an existing render** | ACTIVE — full protocol |
| **User asks for a still image / Midjourney prompt** | DORMANT — use `image-prompt` |
| **Non-video request** | DORMANT — do not activate |

## Engine Reference (deployed)

- **Endpoint**: ComfyUI HTTP API on `dgx-spark` — `http://192.168.181.220:8188` (LAN, **no auth**) or `http://localhost:8188` via `ssh -L 8188:localhost:8188 dgx-spark`. Confirm live: `GET /system_stats`.
- **Model**: Wan2.2-ti2v-5b — text-to-video AND image-to-video. Native **1280×704**, **≤121 frames** (~5s @ 24fps). Files: `wan2.2_ti2v_5B_fp16.safetensors`, `umt5_xxl_fp8_e4m3fn_scaled.safetensors` (CLIP type `wan`), `wan2.2_vae.safetensors`.
- **API**: `POST /prompt` (`{prompt, client_id}`) → `GET /history/{id}` (poll) → `GET /view?filename=..&subfolder=..&type=output` (download). Node defs via `GET /object_info/{Class}`.
- **No audio** — model is video-only. Mux a track in post (ffmpeg, or `VHS_LoadAudio→CreateVideo.audio`).
- **Constraints**: docker on box needs interactive sudo (no headless); ffmpeg lives in the container; outputs persist in the `comfy-output` volume + retrievable via `/view`.
- **OpenMontage bridge (deployed alternative)**: the `comfyui_video` tool inside the `aiplat-openmontage` container drives this same engine over the `aiplat` network (`http://comfyui:8188`) — the submit/poll/download flow wrapped in a `BaseTool` (template-driven graph, fail-loud, `$0`). Prefer it when **composing a montage** via OpenMontage; drive `:8188` directly for raw ad-hoc clips. Bridge tool + workflow JSON live in `dgx-ai-ocgt/deploy/openmontage/`; full design in `dgx-ai-ocgt/docs/14`.

## Protocol

### Step 1: Parse Request
- [ ] Subject, setting, action, mood
- [ ] **Duration** — if >5s, flag beat-splitting (Step 6)
- [ ] **Style** — load the learned style profile (`anime-video-style` in Library / native memory) unless user overrides
- [ ] Reference image? → image-to-video (wire `Wan22ImageToVideoLatent.start_image`)

### Step 2: Check Engine
- [ ] `GET /system_stats` → confirm up + VRAM free
- [ ] If down: surface it, do not fabricate a result

### Step 3: Build Workflow (API format)
- [ ] Assemble the node graph (see Graph Skeleton)
- [ ] Apply style profile to positive + the 3D/CGI negative block
- [ ] Set `length` (≤121), `width/height` (1280×704), `steps` (30), `cfg` (5), `shift` (8), `fps` (24)

### Step 4: Submit + Poll
- [ ] `POST /prompt`; assert `node_errors == {}`
- [ ] Poll `GET /history/{id}` until `status_str == success` (background poller; render ≈ 10-15 min / clip)

### Step 5: Retrieve
- [ ] Read output filename from history, `GET /view` → download to a known path
- [ ] Verify it is a valid MP4 (`file`) before claiming success

### Step 6: Multi-beat (>5s only)
- [ ] Split the storyboard into N beats of ≤5s (length 96 = 4s, 121 = 5s)
- [ ] **Fix character + setting descriptions** and reuse in EVERY beat → cross-cut consistency
- [ ] Independent t2v per beat (fits cut-heavy action) OR i2v-chain last-frame (fits continuous motion) — pick per content, surface the trade-off
- [ ] Queue all beats; one poller downloads each as it lands

### Step 7: Stitch + Audio (multi-beat)
- [ ] ffmpeg concat the beats in order (hard cuts suit anime); optional white-flash transition
- [ ] ffmpeg runs in the container → hand the `sudo docker exec` command to the user (no headless sudo)
- [ ] Mux audio only if the user supplied a track

### Step 8: Confirm
- [ ] Report output path + dims/frames/duration
- [ ] Offer: tune (style/motion/length), more beats, add sound

## Graph Skeleton (Wan2.2 t2v)

```
UNETLoader(wan2.2_ti2v_5B_fp16, default) ─► ModelSamplingSD3(shift 8) ─► KSampler ─► VAEDecode ─► CreateVideo(fps 24) ─► SaveVideo(mp4,h264)
CLIPLoader(umt5..., type wan) ─► CLIPTextEncode(pos) ─► KSampler.positive
                              └► CLIPTextEncode(neg) ─► KSampler.negative
VAELoader(wan2.2_vae) ─► Wan22ImageToVideoLatent(1280,704,length,1) ─► KSampler.latent_image   [+ start_image for i2v]
                      └► VAEDecode.vae
KSampler: seed, steps 30, cfg 5, sampler euler, scheduler simple, denoise 1.0
```

## Style Profiles

**2D hand-drawn anime** (default for anime requests):
- Positive lead: `pure 2D hand-drawn Japanese anime, cel-shaded sakuga animation, flat 2D cel art, traditional anime aesthetic, vibrant anime color grading, dramatic anime lighting, fluid motion`
- Negative: `3D, CGI, Unreal Engine, game graphics, realistic humans, live action, rotoscope, plastic textures, low frame rate, stiff animation, AI morphing, distorted anatomy, extra limbs, bad hands, blurry, watermark, text, photorealistic, western cartoon, chibi, low quality`
- Motion lever: shorter `length` (96) reduces morphing; if too realistic, escalate to an anime LoRA (`LoraLoader`).

**Photoreal / cinematic** (default for realistic / cinematic / live-action requests):
- Positive lead: `photorealistic, cinematic film still, shot on 35mm, shallow depth of field, natural volumetric lighting, physically-based materials, detailed skin and fabric texture, realistic motion, color-graded like a feature film`
- Negative: `anime, cartoon, illustration, drawing, 3D render, CGI, video game, plastic, waxy skin, oversaturated, low detail, blurry, morphing, distorted anatomy, extra limbs, bad hands, watermark, text`
- Settings: `steps 40–50` (more detail), `cfg ~5`, sampler `uni_pc` or `dpmpp_2m` / scheduler `simple` or `karras`; keep `length ≤ 121`. Realism degrades with motion → shorter beats hold detail. Escalate to a realism LoRA if prompting plateaus.

**Profile selection**: pick by request keywords — `realistic / cinematic / photoreal / live-action` → **Photoreal**; `anime / 2D / cartoon / cel` → **2D anime**. If ambiguous, ask. Settings above are **starting points** — validate the look with a short test render and tune per box (the `dgx-ai-ocgt` M6 pass).

**Learned profiles**: read `anime-video-style` (and any other saved profile) from the Library / native memory and apply its keywords + settings. These are refined from user-provided samples — prefer them over the defaults above.

## Mandatory Rules

1. **Never fake a render** — only report a file that exists and passed `file` validation. If the engine is down or the poll times out, say so.
2. **5s ceiling per clip** — never submit `length > 121`; for longer, beat-split. Surface the split.
3. **Fix characters across beats** — reuse identical character/setting text every beat or continuity breaks.
4. **Apply the learned style profile** unless the user overrides; don't silently fall back to defaults if a profile exists.
5. **Audio is post** — never claim sound the model can't produce; mux only a user-supplied track.
6. **Sudo is manual** — print `sudo docker exec` / build commands for the user; never assume headless sudo.
7. **No-auth engine** — never expose `:8188` beyond a trusted LAN; never publish results externally without asking.
8. **Set expectations** — the model can't obey exact camera moves, cuts, or slow-mo; those are prompt nudges + post-editing.
9. **Drive via heredoc, never inline** — when running a pipeline through `docker exec`, pipe a heredoc to python stdin (`sudo docker exec -i <ctr> python3 <<'PY' … PY`), NOT a long `python3 -c "…"`. Long inline strings get newline-wrapped by the terminal → `unterminated string literal` / `SyntaxError` (hit repeatedly 2026-06-22).

## Edge Cases

| Situation | Behavior |
|-----------|----------|
| **Engine returns numpy error** | Image predates the `numpy<2` fix — rebuild container (build bakes the pin) |
| **`CUDA error: operation not permitted` at KSampler** | Poisoned CUDA context (often after a redeploy) — `sudo docker restart comfyui`, then resubmit. Never retry in-place. Beware: `/system_stats` healthcheck stays green while compute is broken (reconfirmed 2026-06-22). |
| **`node_errors` on `POST /prompt`** | Graph node/input mismatch for the running ComfyUI version — fix node names/inputs (probe `GET /object_info/{Class}`), don't resubmit unchanged. |
| **apt/egress fails during build** | ufw FORWARD-drop blocks the docker bridge → build with `--network=host` |
| **>5s requested** | Beat-split; never one-shot |
| **User wants exact lip-sync / audio** | Out of scope for Wan; flag, suggest post or a dedicated model |
| **Too realistic despite 2D prompt** | Escalate to anime LoRA; offer to wire `LoraLoader` |

## Level History

- **Lv.1** — Base: text-to-video + image-to-video over the ComfyUI HTTP API, 2D anime style profile, beat-splitting + stitch for >5s, learned-profile hook. (Origin: 2026-06-15, built from the dgx-spark Wan2.2 deployment + first anime fight-scene renders.)
- **Lv.2** — Deployed bridge + driving discipline: the `comfyui_video` tool in the **openmontage** container (`aiplat-openmontage`, net `aiplat`, in-cluster `comfyui:8188`) wraps submit/poll/download — use it when composing via OpenMontage; raw clips still drive `:8188` directly. Added the heredoc driving rule (9) and the CUDA-poison / healthcheck-blind-spot + `node_errors` edge cases. Workflow graph proven valid on-box (`node_errors=={}`). See `dgx-ai-ocgt/docs/14`. (Origin: 2026-06-22, OpenMontage PATH-B integration + live bridge test.)
