# Prompt Templates

Copy-paste prompts for the image and video steps. Replace `{{CHARACTER}}` with your
subject (example: `orange`). Keep the same subject token across every step.

## Step 2 — Image generation (Pixar / Up style, Q-version)

**Settings**: aspect ratio `1:1`, resolution `1K`, pure white background.
Attach a reference photo of the real subject as `pic1` when available.

Base prompt (as provided):

```
create a pixar style character on a white background. He should be similar style
to a character from Up. he is from a "{{CHARACTER}}" with short hands and legs,
and style like's Q style, you can see pic1 to find some information
```

### Tips for a clean, keyable result
- Add emphasis on the background so keying is easy later:
  `..., isolated on a solid pure #ffffff background, soft even studio lighting,
  no shadows on the background, full body, centered, generous margin`.
- Reinforce the Q-version look:
  `chibi proportions, oversized head, tiny short arms and legs, big expressive eyes,
  smooth glossy 3D render`.
- If the model crops limbs, add `full body visible, do not crop hands or feet`.
- Generate a few variants; pick the one with the cleanest background and most centered pose.

### Example (orange)

```
create a pixar style character on a white background. He should be similar style
to a character from Up. he is from a "orange" (the citrus fruit) with short hands
and legs, and style like's Q style, you can see pic1 to find some information.
Isolated on a solid pure #ffffff background, soft even studio lighting, no
background shadows, full body, centered, generous margin, chibi proportions,
oversized head, tiny short arms and legs, glossy 3D render.
```

Save as `{{character}}-character.png`.

## Step 3 — Image-to-video (idle animation)

Feed the Step 2 image into an image-to-video model. Base prompt (as provided):

```
This character on a solid #fff white background. He's looking around. Static camera
positioning, no panning. Swing your hand gently, not too much, and slowly.
```

### Tips
- Keep it subtle: `subtle idle motion, minimal movement, slow and gentle, seamless loop`.
- Protect the matte: `keep the background solid #ffffff, no camera movement, no motion
  blur, no fast motion, sharp clean edges`.
- Aim for ~3–6s so it can loop.
- If the model drifts the camera, repeat `static locked-off camera, no panning, no zoom`.

Save as `{{character}}-raw.mp4`.

## Reusing the token

| Placeholder | Example value |
|-------------|---------------|
| `{{CHARACTER}}` | `orange` |
| `{{character}}-character.png` | `orange-character.png` |
| `{{character}}-raw.mp4` | `orange-raw.mp4` |
| `{{character}}-role-asset.mov` | `orange-role-asset.mov` |
| `{{character}}-role-asset.webm` | `orange-role-asset.webm` |
