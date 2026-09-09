---
name: 3d-character-generation
description: Produces a Pixar/"Up"-style Q-version 3D animated character as a transparent-background video asset, ready to drop into an app, website, or product UI. Runs the full pipeline: pick a character concept, generate a 1:1 1K Pixar-style image, animate it into a short idle/looking-around clip, key out the white background to an alpha-channel video, then compress it. Trigger when the user mentions "3D character", "3D 角色", "animated mascot", "吉祥物", "Pixar style character", "transparent background video", "透明背景视频", "alpha channel video", "character asset", or wants to turn a product/object into a cute animated 3D mascot.
---

# 3D Animated Character Generation

Turn any concept (a fruit, a product, an animal, an object) into a cute Pixar/"Up"-style
Q-version 3D character, delivered as a **transparent-background (alpha channel) video**
that can be composited directly into an app, landing page, or product UI.

The running example in this skill is an **orange** (橙子) mascot; substitute your own
concept anywhere you see `{{CHARACTER}}`.

## Pipeline Overview

```
1. Concept        → decide the character (e.g. orange)
2. Image gen      → Pixar/Up-style, Q-style, white background, 1:1, 1K
3. Image → video  → idle "looking around + gentle hand swing", static camera, #fff bg
4. Key + export   → remove white background → video WITH alpha channel
5. Compress       → shrink the alpha video for delivery (ffmpeg)
   Output: {{character}}-role-asset  (alpha video, e.g. .webm / .mov)
```

Each step's exact prompts and commands are in `references/prompts.md` and
`references/transparency-and-compression.md`. Read them before running the step.

---

## Step 1 — Choose the character concept

Pick anything: a fruit, a product, a mascot idea. Keep it a single, recognizable
subject so it reads clearly as a Q-version character.

- Example: **orange** (橙子)
- Record it as `{{CHARACTER}}` and use it consistently in every prompt and filename.
- Optional: gather 1–3 reference images (`pic1`, `pic2`, …) of the real subject so the
  image model matches color, texture, and defining features.

Naming convention for the final asset: `{{character}}-role-asset` (lowercase,
kebab-case). Example: `orange-role-asset.webm`.

---

## Step 2 — Generate the 3D-style character image

Use any Pixar-style-capable image model (Midjourney, DALL·E, Nano Banana / Gemini image,
Seedream, etc.). Attach your reference image as `pic1` if you have one.

**Settings**: aspect ratio **1:1**, resolution **1K**, **white background**.

**Prompt template** (see `references/prompts.md` for variants and tips):

```
create a pixar style character on a white background. He should be similar style
to a character from Up. he is from a "{{CHARACTER}}" with short hands and legs,
and style like's Q style, you can see pic1 to find some information
```

Acceptance for this step:
- Clean, evenly lit **pure white** background (critical — it becomes the alpha key).
- Full body in frame with margin on all sides (don't crop hands/feet).
- Clear Q-version proportions: big head, short hands and legs.
- Save as `{{character}}-character.png`.

Regenerate until the background is clean and the pose is centered; a messy or off-white
background makes Step 4 (keying) much harder.

---

## Step 3 — Animate the image into a short video

Feed the Step 2 image into an image-to-video model (Kling, Runway, Hailuo/MiniMax,
Vidu, Veo, Sora, etc.). Keep the motion subtle so the loop reads as a calm idle.

**Prompt template**:

```
This character on a solid #fff white background. He's looking around. Static camera
positioning, no panning. Swing your hand gently, not too much, and slowly.
```

Guidance:
- Keep the **background solid #fff** and the **camera static** — both make keying clean.
- Prefer a short clip (~3–6s) that can loop.
- Avoid fast/large motion or motion blur; it smears the edges and ruins the alpha matte.
- Save as `{{character}}-raw.mp4`.

---

## Step 4 — Remove the white background (export WITH alpha)

Use a video editor to key out the white background and export a format that
**supports transparency**. CapCut (剪映) or DaVinci Resolve both work; DaVinci is more
reliable for true alpha export.

Key points (full walkthrough in `references/transparency-and-compression.md`):
- Apply a **luma/chroma key** (or "3D Keyer" in DaVinci) to knock out the white.
- Refine the matte (clip black/white, slight edge shrink/blur) to remove white fringing.
- Export a format **with an alpha channel**, e.g.:
  - DaVinci: QuickTime **ProRes 4444** (`.mov`) — alpha, high quality, large. **Preferred
    master** because ffmpeg can read its alpha for later compression.
  - Web/app: **WebM VP9 with alpha** (`.webm`).
  - Fallback: **PNG image sequence** (always preserves alpha, universally readable).
- Do **not** export H.264 `.mp4` for this step — standard MP4/H.264 has no alpha and
  will bake the background back in.
- ⚠️ Note on **HEVC-with-alpha** `.mov` (a common macOS "transparent" export): it is real
  alpha and plays in Safari/QuickTime/After Effects, but **ffmpeg cannot decode its alpha
  layer** (it renders transparency as black), so it is a poor master for Step 5
  compression on non-Apple machines. If you plan to compress with ffmpeg, export ProRes
  4444 or a PNG sequence instead. See `references/transparency-and-compression.md`.
- Save as `{{character}}-role-asset.mov` (ProRes 4444) as the master.

> Alpha reality check (ProRes/PNG master): play it over a colored background, or run
> `ffprobe` and confirm the pixel format contains alpha, e.g. `yuva444p10le` / `rgba`.
> If the background is still white, the export format didn't carry alpha. (WebM is
> verified differently — see Step 5's `alpha_mode` note.)

---

## Step 5 — Compress the transparent video (in-agent)

The master (ProRes 4444) is large. Compress it to a delivery-friendly alpha video
using `ffmpeg`, **without losing the alpha channel**. See
`references/transparency-and-compression.md` for the full command set and trade-offs.

Most common target — WebM VP9 with alpha (great for web/app):

```bash
ffmpeg -i {{character}}-role-asset.mov \
  -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 \
  -metadata:s:v:0 alpha_mode=1 -b:v 0 -crf 30 -an \
  {{character}}-role-asset.webm
```

`-auto-alt-ref 0` and `-metadata:s:v:0 alpha_mode=1` are required for VP9 alpha to be
flagged correctly for players (browsers read `alpha_mode`, not the pixel format).

Verify alpha survived compression — for WebM check the **`alpha_mode` tag**, not the
pixel format:

```bash
ffprobe -v error -select_streams v:0 -show_entries stream_tags=alpha_mode -of csv=p=0 \
  {{character}}-role-asset.webm    # expect: 1
```

> Caveat: for a WebM, `ffprobe`'s `pix_fmt` reports `yuv420p` and ffmpeg's own
> RGBA re-decode is unreliable, even when the alpha is intact — VP9 stores alpha as a
> side channel. The authoritative check is `alpha_mode=1` plus previewing in a browser
> over a colored/checkerboard background. (ProRes 4444 and PNG sequences, by contrast,
> do report a `yuva*`/`rgba` pixel format and round-trip cleanly.)

Deliverables:
- `{{character}}-role-asset.mov` — ProRes 4444 master (alpha).
- `{{character}}-role-asset.webm` — compressed VP9 alpha for delivery.

---

## Key Principles

1. **White background is a keying tool** — enforce a clean, pure `#fff` background in
   Steps 2–3 so Step 4's matte is clean. Off-white or shadowed backgrounds cause fringing.
2. **Alpha or it didn't happen** — only formats with an alpha channel (ProRes 4444,
   WebM/VP9 alpha, HEVC-with-alpha, PNG sequence) preserve transparency. Never route the
   transparent asset through H.264 MP4.
3. **Subtle motion** — gentle, slow movement keeps edges crisp and the loop calm.
4. **Verify, don't assume** — after export and after compression, confirm the pixel
   format reports alpha (`ffprobe … pix_fmt` → `yuva*`) and preview over a color.
5. **Consistent naming** — one `{{CHARACTER}}` token drives every filename:
   `{{character}}-character.png` → `{{character}}-raw.mp4` →
   `{{character}}-role-asset.mov` → `{{character}}-role-asset.webm`.
