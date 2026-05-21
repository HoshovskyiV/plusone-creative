# Higgsfield prompt patterns

Templates for the two image-generation paths used by `asset-generator`.

## Path A — Transparent doodles (gpt-image-1.5)

Model: `gpt-image-1.5`. Use case: every doodle layer that must drop onto a Figma frame as an image-fill with real alpha.

### The transparency hard rule

`gpt-image-1.5` defaults to rendering a checkerboard pattern when asked for "transparent background". This is wrong — Figma will display the checkerboard as a fill, not as transparency. Always append this exact clause:

```
Output as PNG with real alpha transparency. The background must be actually transparent with an alpha channel, not a checkerboard pattern, not a white background, not a gradient. No scene, no floor, no wall, no shadow, no glow. Show only the specific object, centered, with small margin of transparent space.
```

Do not paraphrase. Do not shorten. The model needs the explicit list of negatives.

### Full doodle prompt template

```
{concept}, drawn in the style of: {doodle_style from style profile}.

Output as PNG with real alpha transparency. The background must be actually transparent with an alpha channel, not a checkerboard pattern, not a white background, not a gradient. No scene, no floor, no wall, no shadow, no glow. Show only the specific object, centered, with small margin of transparent space.
```

### Higgsfield call

```
generate_image({
  model: "gpt-image-1.5",
  prompt: "<full template>",
  aspect_ratio: "1:1",
  output_format: "png"
})
```

Poll `job_status` until done. If the returned PNG still has a checkerboard pattern (sample center pixel + check edge), reject and re-run once with a stronger negative clause. If it fails twice, surface to user.

## Path B — Photos (nano-banana)

Model: `nano-banana`. Use case: photographic image fills for the photo placeholders in the template.

### Full photo prompt template

```
{photo_concept} — {photo_treatment from style profile}.

Composition: {framing — e.g. medium close-up, centered subject, negative space on right for text overlay}.
Lighting: {soft natural / hard studio / golden hour / overcast — pulled from style profile}.
Color grading: {warm / cool / desaturated / vivid — pulled from style profile}.
Background: {clean / environmental / cut-out studio — pulled from style profile}.

Photorealistic, sharp focus, no text in image, no logos, no watermarks.
```

### Higgsfield call

```
generate_image({
  model: "nano-banana",
  prompt: "<full template>",
  aspect_ratio: "<derive from template frame: 1:1, 4:5, 9:16>"
})
```

## Parallel batching

`asset-generator` should fire all `2N` calls (N doodles + N photos) in parallel — do not sequence. Use `job_status` polling to gather results. If the Higgsfield account balance is below the projected job cost, stop before starting and surface the count.

## Cost guardrails

- gpt-image-1.5: roughly 1 credit per generation. 20 doodles = 20 credits.
- nano-banana: roughly 1 credit per generation. 20 photos = 20 credits.
- For `N > 20`, confirm with the user that the cost is acceptable before firing.

## Failure recovery

| Symptom | Cause | Fix |
|---|---|---|
| Checkerboard in doodle PNG | model ignored alpha clause | re-run once with appended `IMPORTANT: alpha channel must be real, not visual checkerboard pattern` |
| Doodle has unwanted background scene | prompt too literal | strip background descriptors, reinforce `no scene, no floor` |
| Photo has text or watermarks | model hallucination | re-run with stronger negatives `no text whatsoever, no signs, no letters` |
| Photo aspect wrong | aspect_ratio param wrong | re-derive from template frame's actual ratio |
| All jobs failed | Higgsfield outage or auth | check `balance` tool; if 0, surface; if works, retry |
