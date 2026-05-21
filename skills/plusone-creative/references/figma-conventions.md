# Expected Figma template conventions

What `plusone-creative` assumes about a well-formed Plusone template. When a project violates these, warn but proceed.

## A valid template frame

The reference frame the `style-analyzer` returns as `template_node_id` should satisfy:

### Structural

- A single Frame node with auto-layout enabled (vertical or horizontal flow).
- Aspect ratio matches a known IG format: 1080×1080 (square), 1080×1350 (portrait), 1080×1920 (story).
- Contains both at least one text layer **and** at least one image-fill placeholder.

### Naming

Layers named in English, kebab- or camel-case. Expected names (case-insensitive substring match):

| Slot | Expected layer name contains |
|---|---|
| Headline | `headline`, `title`, `h1`, `heading` |
| Subline | `subline`, `subtitle`, `subhead`, `caption` |
| Body / description | `body`, `description`, `text` |
| CTA | `cta`, `button`, `link` |
| Photo / image | `photo`, `image`, `picture`, `cover`, `bg` |
| Doodle | `doodle`, `illustration`, `sticker`, `decoration` |
| Logo | `logo`, `brand-mark`, `wordmark` |

Plusone templates conventionally lead with the section role (`photo-cover`, `doodle-accent`), not visual descriptors (`circle-thing`).

### Components

The template should reuse the project's components and not redraw primitives. Specifically:

- The logo is an **instance** of a `Logo` component, never a flat copy.
- Buttons are instances of a `Button` component with variants for state/size.
- Recurring decorative elements (doodles used across many posts) are components with variants for color/orientation.

If a "template" frame contains raw vectors instead of component instances, treat it as a degraded template — write to it anyway but flag in the final report that the project lacks a real design system.

### Variables and styles

- Colors come from a variables collection (`color/brand/primary`, `color/neutral/100`, etc.). Raw hex fills are a red flag.
- Text uses named text styles (`H1`, `Body`, `CTA`). No detached overrides.
- Spacing comes from variables (`spacing/sm`, `spacing/md`) or consistent auto-layout gap values.

When binding text in `frame-composer`, **never** override font/size/weight inline. Always set the text content only and let the named style do the rest. Inline overrides are the most common cause of validator failure.

## When the project does not follow these conventions

Two graceful degradations:

### Soft warning, proceed
Project has the right structure but loose naming (e.g. `Untitled-frame-23` instead of `post-template`). Proceed using positional heuristics (top-most text = headline, largest image-fill = photo). Add to `warnings` in the style profile.

### Hard stop
Project has no usable template frame, or every "template" is a flattened image. Stop the workflow and ask the user:

> Не знайшов готового шаблону для постів у цьому файлі. Дай URL конкретного фрейму, який треба брати за зразок (правий клік → Copy link).

## Image-fill placeholder shape

Photo placeholders are rectangles with:
- An image fill (any placeholder PNG / solid gray / current photo) — never an empty fill.
- A clip-content boundary matching the desired display crop.

Doodle placeholders are usually:
- A rectangle or frame with an image fill that holds the current doodle PNG.
- Positioned with auto-layout so it stays anchored when content length changes.

When writing, replace the existing image-fill — do not insert a new node alongside.

## Naming the duplicated frames

When `frame-composer` duplicates the template `N` times, name each clone as `<template-name>-<NN>` (e.g. `post-template-01`, `post-template-02`). Place all clones on a dedicated page named `Generated — <YYYY-MM-DD>` so the designer can find them and the original template stays untouched.
