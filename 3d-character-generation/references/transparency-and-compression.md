# Transparency & Compression

Steps 4–5 turn the white-background clip into a compact **alpha-channel** video.
The one rule that governs everything: **only alpha-capable formats keep transparency.**

## Alpha-capable formats (pick your delivery target)

| Format | Container | ffmpeg codec | Alpha pixel fmt | Use for |
|--------|-----------|--------------|-----------------|---------|
| ProRes 4444 | `.mov` | `prores_ks` | `yuva444p10le` | Editing master, highest quality (large) |
| VP9 + alpha | `.webm` | `libvpx-vp9` | `yuva420p` | Web / app delivery (best size) |
| VP8 + alpha | `.webm` | `libvpx` | `yuva420p` | Legacy web fallback |
| HEVC + alpha | `.mov` | `hevc_videotoolbox` (macOS) | `bgra`→alpha | Apple / iOS / Safari |
| PNG sequence | `%04d.png` | `png` | `rgba` | Universal fallback, always works |

> Formats that **cannot** hold alpha: H.264/AVC `.mp4`, most `.mp4` in general, GIF
> (1-bit "transparency" only). Never route the transparent asset through H.264 MP4 —
> it bakes the background back in.

## Step 4 — Key out the white background

### DaVinci Resolve (recommended for true alpha)
1. Drop `{{character}}-raw.mp4` on the timeline.
2. Color page → add a **3D Keyer** (or **Qualifier**) node; sample the white background.
3. Refine the matte: adjust **Clean Black / Clean White**, add a small **Blur** and
   **Shrink** to remove white fringing around the edges.
4. Add an **Alpha Output** node and connect the key to it (so the removed area becomes
   transparent, not black).
5. Deliver page → Format **QuickTime**, Codec **ProRes 4444**, and check
   **Export Alpha**. Output `{{character}}-role-asset.mov`.

### CapCut / 剪映
1. Import the clip, apply **Chroma key / 抠像** and pick the white background color;
   raise **Intensity/强度** and **Shadow/阴影** until the background is gone.
2. Note: CapCut's standard exports are H.264 MP4 (no alpha). To keep transparency,
   export a **PNG sequence** (or use the desktop pro alpha export if available), then
   convert with ffmpeg below. If you only need a colored-background composite, you can
   place your own background layer under the keyed character instead.

### Sanity check the matte
Preview the keyed clip over a bright color (magenta/green). Any white halo means the
matte needs more edge shrink/blur, or the source background wasn't clean enough — if so,
regenerate Step 2/3 with a cleaner `#fff` background.

## Step 5 — Compress with ffmpeg (keep alpha)

### WebM VP9 + alpha (primary delivery)
```bash
ffmpeg -i {{character}}-role-asset.mov \
  -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 \
  -metadata:s:v:0 alpha_mode=1 -b:v 0 -crf 30 -an \
  {{character}}-role-asset.webm
```
- `-auto-alt-ref 0` + `-metadata:s:v:0 alpha_mode=1`: **required** so the alpha is
  flagged for players. Browsers key off the `alpha_mode` tag, not the pixel format.
- `-crf` 28–34: lower = higher quality/larger. Start at 30.
- `-b:v 0` enables constant-quality (CRF) mode.
- `-an` drops audio (idle asset needs none).

### Smaller / two-pass VP9 (optional)
```bash
ffmpeg -i in.mov -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 \
  -metadata:s:v:0 alpha_mode=1 -b:v 0 -crf 32 -row-mt 1 \
  -deadline good -cpu-used 2 -an out.webm
```

### From a PNG sequence (CapCut path)
```bash
ffmpeg -framerate 30 -i frames/%04d.png \
  -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 \
  -metadata:s:v:0 alpha_mode=1 -b:v 0 -crf 30 -an out.webm
```

### ProRes 4444 master from a PNG sequence (if you need a .mov master)
```bash
ffmpeg -framerate 30 -i frames/%04d.png \
  -c:v prores_ks -profile:v 4444 -pix_fmt yuva444p10le out.mov
```

### HEVC + alpha (Apple platforms, macOS only)
```bash
ffmpeg -i in.mov -c:v hevc_videotoolbox -alpha_quality 0.9 -tag:v hvc1 out_hevc.mov
```

## Always verify alpha survived

The correct check **depends on the format** — this trips people up.

### WebM (VP9/VP8): check the `alpha_mode` tag, NOT the pixel format
```bash
ffprobe -v error -select_streams v:0 -show_entries stream_tags=alpha_mode -of csv=p=0 out.webm
# expect: 1
```
For a WebM, `ffprobe` reports `pix_fmt=yuv420p` and ffmpeg's own RGBA re-decode looks
like the background went opaque **even when the alpha is fine** — VP9 stores alpha as a
side channel that ffmpeg's self-decode doesn't reconstruct reliably. The authoritative
proof is `alpha_mode=1` **plus** previewing in a browser over a colored/checkerboard
background (browsers honor the alpha correctly).

### ProRes 4444 / PNG sequence: check the pixel format
```bash
# These DO round-trip cleanly; pixel format must be a yuva*/rgba variant:
ffprobe -v error -select_streams v:0 -show_entries stream=pix_fmt -of csv=p=0 master.mov
# expect: yuva444p10le (ProRes) / rgba (PNG)
```

### Format-agnostic functional proof
Composite over a bright color and confirm the color shows through the background:
```bash
ffmpeg -f lavfi -i color=c=magenta:s=256x256 -i out.webm \
  -filter_complex "[0][1]overlay=eof_action=endall" -frames:v 1 check.png
# open check.png — background around the character must be magenta, not black
```

If verification fails, re-export Step 4 in an alpha format (ProRes 4444 / PNG sequence)
and recompress.

## Quick decision guide

- Web page / in-app animation → **WebM VP9 alpha** (`.webm`).
- Apple/iOS-only surface → **HEVC alpha** (`.mov`).
- Need to re-edit later → keep the **ProRes 4444** master.
- Tooling can't emit alpha video → **PNG sequence**, then convert with ffmpeg.
