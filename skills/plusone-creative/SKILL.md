---
name: plusone-creative
description: Autonomous Instagram creative generation for designers. Extracts visual style (doodles, photos, palette, typography) from a referenced Figma project, generates assets via Higgsfield, and composes frames in Figma using the project's existing design system. Use when the user asks to create, adapt, batch, or restyle Instagram posts, carousels, banners, or social-media creatives in the style of a specific Figma file. Triggers (UA + EN) — інстаграм пост, інстаграм креатив, креатив для клієнта, адаптуй шаблон, новий банер, згенеруй пост у стилі проекту, зроби N постів, пости у стилі проекту, банер у стилі бренду, інстаграм-карусель, креатив у Figma, пост для соцмереж, адаптуй під клієнта, витягни стиль з Figma, дудли у стилі проекту, фото у стилі бренду, батч креативів, instagram post, instagram carousel, social media creative, brand-style post.
---

# Plusone Creative

Autonomous workflow that turns a brief + a Figma project URL into a batch of finished Instagram creatives, fully matched to that project's visual identity.

## Mandatory dependencies

Before any write operation in Figma, load these skills:

- `figma-use` — required reading for all `use_figma` calls. Covers Plugin API patterns, auto-layout, variable bindings, error recovery. **Never call `use_figma` without it.**
- `figma-generate-design` — workflow for translating descriptions into Figma frames using existing components, styles, and variables.

For asset generation reference `references/prompt-patterns.md`. For style extraction reference `references/style-extraction.md` and `references/doodle-deconstruction.md`. For template assumptions reference `references/figma-conventions.md`.

## Tools required

- Figma MCP: `get_design_context`, `get_screenshot`, `use_figma` (read + write).
- Higgsfield MCP: `generate_image` (gpt-image-1.5 for transparent doodles, nano-banana for photos), `media_upload`, `media_confirm`, `job_status`.

If either MCP is not connected, stop and tell the user to enable the connector in Claude Desktop → Customize → Connectors.

## Workflow — 10 steps

### 1. Parse the user brief
Identify: number of creatives `N`, target format (square 1080×1080, story 1080×1920, carousel), language of copy, deadline/urgency, any explicit style notes.

### 2. Identify the Figma project URL
The user must provide a Figma file URL or node URL (e.g. `https://www.figma.com/file/...` or `https://www.figma.com/design/...?node-id=...`). If missing, ask once: "Який Figma-проект береш за стилевий референс? Дай URL файлу або конкретного фрейму."

### 3. Invoke `style-analyzer` sub-agent
Delegate full style inventory to the `style-analyzer` agent. It returns a structured JSON style profile:
```json
{
  "doodle_style": "...",
  "photo_treatment": "...",
  "palette": ["#...", "#..."],
  "typography_voice": "...",
  "template_node_id": "...",
  "notable_components": ["..."]
}
```
Keep this profile in working memory for all downstream steps. Do not re-extract style mid-batch.

### 4. Parse content brief
Content source is one of:
- Inline text from the user.
- A file path (`.md`, `.txt`, `.csv`, `.xlsx`) — read it.
- A Google Doc / Drive link — use the Drive connector to fetch.

Extract `N` content units. Each unit must yield: a headline, a subline (optional), one doodle concept, one photo concept.

### 5. Build the content matrix
Construct an `N × 4` table in working memory:

| # | headline | subline | doodle_concept | photo_concept |
|---|----------|---------|----------------|---------------|

Show this matrix back to the user before generating any asset. Wait for confirmation if the user asked for review; otherwise proceed.

### 6. Invoke `asset-generator` sub-agent (parallel)
Pass the matrix + the style profile to `asset-generator`. It runs Higgsfield calls in parallel — one job per doodle, one per photo. Doodles use `gpt-image-1.5` with the hard transparent-PNG prompt from `references/prompt-patterns.md`. Photos use `nano-banana` with the photo treatment description from the style profile. Returns `2N` finished media URLs (one doodle + one photo per creative).

### 7. Invoke `frame-composer` sub-agent
Pass the matrix + asset URLs + `template_node_id` to `frame-composer`. It:
- Duplicates the template frame `N` times in the Figma file.
- Sets text in the existing text layers (using named text styles, never inline overrides).
- Fills the image placeholders via image-fill.
- Drops the doodle PNG into the doodle slot.
- Reuses existing components, variables, and styles. Never draws raw primitives.

### 8. Invoke `validator` sub-agent
Pass the list of composed frame IDs to `validator`. It screenshots each frame and checks:
- No text overflowing its container.
- Doodle not occluding logo or safe-zone areas.
- All image-fills populated (no grey rectangles).
- Named text styles applied (no detached overrides).
- Palette consistent with the style profile.

Returns a per-frame status: `ok` or `{frame_id, issue, suggested_fix}`.

### 9. Retry loop (if needed)
For each `validator` issue:
- Text overflow → invoke `frame-composer` with a shorter copy variant or smaller text style.
- Doodle placement → re-position via `use_figma`.
- Missing image-fill → re-upload or re-generate the photo.
- Detached style → reapply the named style.

Max 2 retry rounds per frame. If still failing, flag for manual review in the final report.

### 10. Report back to the user
Output in Ukrainian:
- Скільки креативів готово, скільки потребують ручної доводки.
- Прямі лінки на готові фрейми у Figma (`https://www.figma.com/file/...?node-id=...`).
- Список фрейми → проблеми, які треба дотягнути руками.
- Підказка: де натиснути в Figma щоб подивитись готовий batch.

## Hard rules

- **Never** invent style attributes. If the style-analyzer cannot determine something, leave it blank and warn the user.
- **Never** draw doodles or photos as Figma primitives — always generate via Higgsfield and place as image-fills.
- **Never** hardcode colors or fonts — always bind to the project's variables and named styles.
- **Never** call `use_figma` without loading the `figma-use` skill first in the same turn.
- For doodles requiring transparent background, use the exact transparency prompt from `references/prompt-patterns.md` — do not paraphrase.

## When to stop and ask

- Figma file URL missing or returns 403 (user not authorized).
- Higgsfield balance insufficient — surface the credit count.
- Template frame not recognizable (no clear text slots / image placeholders). Show the user what was found and ask for the correct node URL.
- `N > 20` — confirm with the user, since this will be expensive.
