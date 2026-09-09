# Example — Orange (橙子) Mascot

End-to-end run of the pipeline with `{{CHARACTER}} = orange`.

| Step | Prompt / action | Output |
|------|-----------------|--------|
| 1. Concept | Character = an orange (citrus fruit) | `orange` |
| 2. Image | "create a pixar style character on a white background … from an \"orange\" … Q style … pic1" — 1:1, 1K, white bg | `orange-character.png` |
| 3. Video | "This character on a solid #fff white background. He's looking around. Static camera positioning, no panning. Swing your hand gently, not too much, and slowly." | `orange-raw.mp4` |
| 4. Key + export | Key out white bg, export a video WITH alpha | `orange-health-role-asset.mov` |
| 5. Compress | compress the alpha video (see caveat below) | `orange-health-role-asset.webm` |

The committed master `orange-health-role-asset.mov` is the real deliverable produced by
this pipeline (stored via Git LFS).

## The committed master (verified)

`orange-health-role-asset.mov` — a genuine **transparent-background** clip:

- Codec: **HEVC / `hvc1`**, 960×960, 60 fps, ~10 s.
- Alpha: **yes** — the file carries an HEVC auxiliary alpha layer
  (`nuh_layer_id = 1`) plus the alpha-channel-info SEI (`payloadType 165`). This is the
  "HEVC with alpha" format that Apple/QuickTime/Safari and After Effects understand.

### ⚠️ Caveat: ffmpeg cannot transcode HEVC-alpha

Most command-line tooling — including `ffmpeg` — decodes only the HEVC **base** layer and
ignores the alpha layer, rendering the transparent region as **solid black**. So you
cannot generate the compressed `orange-health-role-asset.webm` from this HEVC master with
`ffmpeg` on a generic (e.g. Linux) box, and you must NOT "key out" the black — the
character's eyes are black and would be punched out.

To produce the web-delivery `*.webm` (VP9 + alpha), use one of:

1. **On macOS** (where HEVC-alpha decodes), convert directly:
   ```bash
   ffmpeg -i orange-health-role-asset.mov \
     -c:v libvpx-vp9 -pix_fmt yuva420p -auto-alt-ref 0 \
     -metadata:s:v:0 alpha_mode=1 -b:v 0 -crf 30 -an \
     orange-health-role-asset.webm
   ```
2. **Anywhere** — first export an ffmpeg-friendly alpha master from your editor
   (**ProRes 4444** `.mov` or a **PNG sequence**), then run the same VP9 command. This is
   the most portable path and is why the skill recommends ProRes 4444 / PNG sequence as
   the master for compression (see `../references/transparency-and-compression.md`).

## Verifying alpha (format-specific)

```bash
# HEVC-with-alpha (this master): ffmpeg reports yuv420p, so check the bitstream instead —
# look for an auxiliary layer (nuh_layer_id 1) and the alpha SEI (payloadType 165),
# or use a tool like `mediainfo` which prints "Alpha: Yes".

# ProRes 4444 / PNG: pixel format itself carries alpha
ffprobe -v error -select_streams v:0 -show_entries stream=pix_fmt -of csv=p=0 master.mov
# expect yuva444p10le (ProRes) / rgba (PNG)

# WebM (VP9/VP8): check the alpha_mode tag, not the pixel format
ffprobe -v error -select_streams v:0 -show_entries stream_tags=alpha_mode -of csv=p=0 out.webm
# expect 1
```

## Adding the binary to the repo (Git LFS)

Video binaries are tracked via Git LFS (`.gitattributes` at the repo root):

```bash
git lfs install
git add 3d-character-generation/examples/orange-health-role-asset.mov
git commit -m "Add orange mascot example asset (LFS)"
```
