# Style extraction from a Figma project

How to inventory a Figma file's visual identity so the rest of the workflow can reproduce it.

## Entry points

Given a Figma file or node URL, call `get_design_context` on the root of the file or on the most representative frame. Look for:

- **Components and component sets** — anything in the Assets panel; names like `Card`, `Post`, `Story`, `Banner`, `Doodle`, `Avatar`, `Logo`.
- **Variables collections** — color tokens (`color/brand/*`, `color/accent/*`), spacing, radius, opacity.
- **Text styles** — named entries like `H1`, `H2`, `Body`, `Caption`, `CTA`.
- **Effect styles** — shadows, blurs.
- **Existing finished creatives** — frames already designed in the file are the truest reference, more than the component library.

## What to inventory

Build four buckets in working memory:

### 1. Doodle inventory
Find every layer that is hand-drawn / illustrative — usually image fills (PNG/SVG) named with words like `doodle`, `illustration`, `sticker`, `decoration`, `squiggle`, `arrow`, `circle`, or vector groups with non-geometric paths. Take 3–5 representative screenshots via `get_screenshot`. These feed the doodle-deconstruction step.

### 2. Photo inventory
Find every image-fill rectangle that holds a photograph. Sample 3–5 via `get_screenshot`. Note: aspect ratios, subject types (people / product / abstract), color grading (warm/cool/desaturated/vivid), background treatment (clean studio / environmental / cut-out).

### 3. Palette
Read the variables collection. Fall back to sampling fill colors from existing frames if no variables exist. Output as `["#RRGGBB", ...]` ordered by frequency (primary → accent → neutral).

### 4. Typography voice
Read text styles. Capture: typeface family names, weights, size hierarchy, letter-spacing, case (uppercase/title/sentence). Voice descriptor: `editorial-serif`, `geometric-sans-loud`, `humanist-friendly`, `display-condensed`, etc.

## Template detection

Find the frame that looks most like a posting template — usually:
- Aspect ratio matches the target format (1080×1080, 1080×1920, 1080×1350).
- Has both text layers and image-fill placeholders.
- Named `Template`, `Post template`, `Story template`, `IG post`, or similar.
- Often in a page called `Templates`, `Production`, `Posts`, `IG`.

Capture its `node_id`. This becomes `template_node_id` in the style profile.

## Output format

Always emit a single JSON object — no extra prose:

```json
{
  "doodle_style": "one-sentence description from doodle-deconstruction.md vocabulary",
  "photo_treatment": "one-sentence description: subjects, grading, framing",
  "palette": ["#RRGGBB", "#RRGGBB", "..."],
  "typography_voice": "voice descriptor + primary typeface",
  "template_node_id": "1:234",
  "notable_components": ["Card", "Doodle/Squiggle", "..."],
  "warnings": ["..."]
}
```

If any bucket is empty or unclear, populate `warnings` instead of guessing.

## Common failure modes

- **Empty file** — file URL valid but no usable content. Stop and ask the user for a more populated reference.
- **Mixed brands in one file** — multiple unrelated clients in the same Figma. Ask the user which page / section is the reference.
- **No variables, no text styles** — older file with raw values. Sample by hand from a finished frame; flag in `warnings` that the project lacks tokens.
- **All doodles are flattened raster** — cannot inspect vector paths. Rely on `get_screenshot` of the doodle layer rather than tree traversal.
