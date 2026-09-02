---
name: MiniMax H3 吹哨舞换人
description: >-
  use this when swapping a new character onto the locked MiniMax H3 吹哨舞
  (source_motion.mp4) Ref2VA recast — 三视图 in (keep face: split portrait +
  full-body), no new reference video
---
# MiniMax H3 吹哨舞换人

Same white-studio whistle-dance performance, new character. Do **not** ask for a new reference video. Source is already on Comfy as `source_motion.mp4` (~6.2 s, 720×1280, 30 fps, high-angle white cyc, original green-slip performer, hair-whip / sway, EDM). Keep camera, timing, motion, and audio. Swap only the person.

If the user brings a **different** dance/camera/cut, stop and run [MiniMax H3 R2V](sand-workflow:minimax-h3-r2v) instead.

Write prompts in English. Talk to the user in 简体中文. Local ComfyUI only (H3 Ref2VA + RIFE + UltraSharp). No cloud MCP.

## Need from the user
- Character 三视图 (portrait + front/side/back). **Keep the face.** Split the sheet: close-up portrait = identity; full-body three views = outfit/body. Do not feed the giant headshot as the only image (it pulls composition to center), and do not throw the face away.
- Optional: they already confirmed a first-frame still.

Do not wait for a new video.

## Defaults
- Native H3: **24 fps**, **0.8 MP**, **9:16** → **672×1216**, length ~**158 frames / 6.58 s**
- **Always** confirm a first-frame still before full R2V
- **Always** RIFE **3× → 72 fps** (`rife_v4.26.safetensors`)
- Upscale **only if asked**: `4x-UltraSharp.pth`, 40-frame chunks, concat to **1440×2560**. Never 4× the whole clip in one batch (OOM ~46 GB)

## Weights / nodes
- UNET `minimax_h3_ref2va_pruned_int8_convrot.safetensors`
- CLIP `qwen3vl_32b_h3_ultra_uncensored_heretic_int8_convrot.safetensors`
- VAE `minimax_h3_video_vae_fp16` / `minimax_h3_audio_vae_fp32`
- `MiniMaxH3ReferenceToVideo` + still via `MiniMaxH3StillConditioningT8` `direct_1_frame`
- Turbo/Lightning LoRA off unless asked
- Templates beside this skill: `r2v_workflow.json`, `rife_3x_api.json`, `upscale_chunk_api.json`

## Pipeline
1. Split the sheet: `character_portrait.png` (left close-up) + `character_fullbody.png` (front/side/back). Ignore red marks on a panel. Confirm the split with the user if they care about face.
2. 1-frame still: source first frame = composition (`edit_image`); portrait = face ref; full-body = outfit ref. Show it. Iterate until the person is on the **same side** as the source, exactly one body, no green-dress ghost, face matches the portrait. Lock as `confirmed_first_frame.png`.
3. Queue R2V: confirmed still + portrait + full-body + `source_motion.mp4`. Prompt: one person; face from portrait; outfit from full-body; white cyc and motion from the source; opening is the confirmed still; reuse EDM.
4. RIFE 3×: LoadVideo → GetVideoComponents → FrameInterpolate multiplier=3 → CreateVideo fps=72.
5. If they want 1.5K: `VHS_LoadVideo` format=None, cap=40, skip=40*i; UltraSharp batched per_batch=1 float16 downscale 0.54; ImageScale 1440×2560; concat; mux source audio. OOM → retry cap=20.

## Output
24 fps native + 72 fps interpolated always; 1440p only when asked. Give Comfy `output/video/` name and a chat-playable copy.
