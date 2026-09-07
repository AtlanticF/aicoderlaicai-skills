# Example — Orange (橙子) Mascot

End-to-end run of the pipeline with `{{CHARACTER}} = orange`.

| Step | Prompt / action | Output |
|------|-----------------|--------|
| 1. Concept | Character = an orange (citrus fruit) | `orange` |
| 2. Image | "create a pixar style character on a white background … from an \"orange\" … Q style … pic1" — 1:1, 1K, white bg | `orange-character.png` |
| 3. Video | "This character on a solid #fff white background. He's looking around. Static camera positioning, no panning. Swing your hand gently, not too much, and slowly." | `orange-raw.mp4` |
| 4. Key + export | Key out white bg in DaVinci/CapCut, export ProRes 4444 with alpha | `orange-role-asset.mov` |
| 5. Compress | `ffmpeg -i orange-role-asset.mov -c:v libvpx-vp9 -pix_fmt yuva420p -b:v 0 -crf 30 -an orange-role-asset.webm` | `orange-role-asset.webm` |

## Final asset

The finished transparent-background clip is `orange-health-role-asset.mov`
(ProRes 4444 / alpha master). Place it in this directory.

### Adding the binary to the repo

The master `.mov` is a large binary. Two options:

1. **Recommended — Git LFS** (keeps the repo lean):
   ```bash
   git lfs install
   git lfs track "3d-character-generation/examples/*.mov"
   git lfs track "3d-character-generation/examples/*.webm"
   git add .gitattributes 3d-character-generation/examples/orange-health-role-asset.mov
   git commit -m "Add orange mascot example asset (LFS)"
   ```
2. **Plain commit** (only if the file is small): `git add` the file directly.

> Tip: also commit the compressed `orange-health-role-asset.webm` (VP9 alpha) — it is
> far smaller and is what most consumers should actually embed.

## Verifying the asset carries alpha

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=pix_fmt -of csv=p=0 \
  orange-health-role-asset.mov     # expect a yuva* / rgba / bgra pixel format
```
