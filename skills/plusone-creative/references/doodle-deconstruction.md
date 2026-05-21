# Doodle deconstruction

How to translate a visual doodle into a precise verbal description suitable as input to an image-generation prompt.

A doodle description has six axes. Always answer all six.

## The six axes

### 1. Line weight
How thick are the strokes? Choose one: `hairline (≈1px)`, `thin (2–3px)`, `medium marker (4–6px)`, `bold brush (7–12px)`, `chunky crayon (≥12px)`, `variable (calligraphic, tapering)`.

### 2. Fill style
Is the shape filled or just outlined? Choose: `outline only, no fill`, `solid fill, no outline`, `outline + solid fill`, `outline + cross-hatch fill`, `outline + scribble fill`, `flat color blocks`, `dotted texture fill`.

### 3. Color treatment
How is color used? Choose: `single accent color (specify hex)`, `two-color (brand + neutral)`, `multi-color palette pulled from brand`, `monochrome black/white`, `monochrome on transparent`, `gradient fill`.

### 4. Abstraction level
How literal is the doodle? Choose: `pictographic (clearly recognizable object)`, `semi-abstract (object hinted, simplified)`, `gestural (pure mark-making — squiggle, arrow, scribble)`, `geometric (composed of basic shapes)`, `symbolic (icon-like)`.

### 5. Texture
What is the surface quality? Choose: `clean vector (no texture)`, `marker bleed (slight blur at edges)`, `pencil grain`, `crayon texture`, `paint stroke (visible brushwork)`, `digital glitch`, `risograph dotted`.

### 6. Geometric vs organic
Choose: `strictly geometric (compass-and-ruler)`, `loose organic (hand-drawn wobble)`, `mixed`.

## Five formulation examples

These are ready-to-use doodle-style strings. Pick the closest, then adapt:

1. `Hand-drawn marker outlines in single brand accent color, no fill, medium weight (4px), loose organic squiggle, slight marker bleed at edges, gestural and abstract.`

2. `Bold black brush strokes on transparent background, 8px chunky weight, solid no outline, monochrome, semi-abstract pictograms with visible paint texture.`

3. `Thin clean vector outlines, 2px hairline, no fill, single accent color (brand red), strictly geometric, no texture, symbolic icon style.`

4. `Crayon-textured doodle with cross-hatch fill, two-color (brand primary + soft cream), variable weight 3–7px, organic and pictographic with visible crayon grain.`

5. `Risograph-style dotted fill with hand-drawn outline, multi-color pulled from brand palette, medium 5px outline, semi-abstract gestural shapes with halftone dotted texture.`

6. `Calligraphic ink stroke with tapering line weight, single warm accent color, no fill, pure gestural mark-making — arrows, swooshes, underlines, organic.`

7. `Flat geometric color blocks with no outline, three-color brand palette, fully filled simple shapes (circle / triangle / squiggle), zero texture, modernist abstract.`

## Mapping to image-gen

When sending to `generate_image` (Higgsfield), the doodle-style string becomes part of the prompt. Concatenate as:

```
<concept>, drawn in the style of: <doodle-style string>. <transparency clause from prompt-patterns.md>.
```

Example concept: `a coffee cup with steam`.

Full prompt:
```
A coffee cup with steam, drawn in the style of: hand-drawn marker outlines in single brand accent color, no fill, medium weight, loose organic squiggle, slight marker bleed, gestural. Output as PNG with real alpha transparency...
```

## Common mistakes

- Skipping the texture axis — image gens default to `clean vector` if not told otherwise; brand doodles often have hand-drawn texture.
- Using vague color descriptors (`brand colors`) when the prompt needs hex values or named colors.
- Mixing pictographic intent with gestural style — the model will pick one and disappoint. Decide upfront.
- Forgetting line weight — `thin doodle` and `chunky doodle` are very different visual genres.
