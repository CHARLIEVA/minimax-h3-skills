---
name: MiniMax H3 R2V
description: >-
  use this when doing a new MiniMax H3 Ref2VA recast: user must provide both a
  reference video and a character turnaround, then 24fps generate → RIFE 3x →
  optional chunked upscale
---
# MiniMax H3 R2V

New Recast: keep a **user-supplied** reference video's camera/timing/motion, replace the performer with a **user-supplied** character turnaround. Both inputs are required. If either is missing, ask — do not reuse `source_motion.mp4` or an old 三视图.

For the locked white-studio whistle dance (same clip, new person only) use [MiniMax H3 吹哨舞换人](sand-workflow:minimax-h3) instead.

Write prompts in English. Talk to the user in 简体中文. Local ComfyUI only (H3 Ref2VA + RIFE + UltraSharp). No cloud MCP.

## Need from the user (block until you have both)
1. **参考视频** — the motion/camera/audio to keep
2. **参考人物** — 三视图 or equivalent identity sheet

## Defaults
- Native H3: **24 fps**, **0.8 MP**, **9:16** → **672×1216**
- Length: match the source; apply the local H3 length formula (24 fps, snap to the 17-frame grid)
- **Always** confirm a first-frame still before full R2V
- **Always** RIFE **3× → 72 fps** (`rife_v4.26.safetensors`). Integer multipliers only. Never CreateVideo 60 fps on native 24 fps frame counts (that shortens the clip)
- Upscale **only if asked**: `4x-UltraSharp.pth`, 40-frame chunks, concat to **1440×2560**. Never 4× the whole clip in one `ImageUpscaleWithModelBatched`

## Weights / nodes
- UNET `minimax_h3_ref2va_pruned_int8_convrot.safetensors`
- CLIP `qwen3vl_32b_h3_ultra_uncensored_heretic_int8_convrot.safetensors`
- VAE `minimax_h3_video_vae_fp16` / `minimax_h3_audio_vae_fp32`
- `MiniMaxH3ReferenceToVideo` (not I2V / FL2VA)
- Still: `MiniMaxH3StillConditioningT8` `direct_1_frame`
- Turbo/Lightning LoRA off unless asked
- RIFE: `models/frame_interpolation/rife_v4.26.safetensors`
- Upscale: `models/upscale_models/4x-UltraSharp.pth`
- Templates: `r2v_workflow.json`, `rife_3x_api.json`, `upscale_chunk_api.json`

## Pipeline
0. Upload 参考视频 (new filename, not necessarily `source_motion.mp4`). Crop 三视图: **drop the giant headshot**, keep full-body (portrait leaks to center + ghost). Ignore red marks.
1. 1-frame still: source frame 0 for composition, sheet for identity. Show it. Iterate until same-side placement, one body, no leftover original costume ghost. Lock `confirmed_first_frame.png`.
2. R2V at 0.8 MP 24 fps. Prompt locks one person, source camera/scene/motion, identity from the sheet, opening from the still, source audio.
3. Interpolate multiplier=3, CreateVideo 72 fps, keep duration.
4. Upscale only on request: VHS load cap=40, UltraSharp batched, scale to 1440×2560, concat, mux audio. Chunk OOM → cap=20.

## Output
24 fps + 72 fps always; 1440p only when asked. Comfy `output/video/` name + chat copy.
