# Strength Hack Slide Deck Converter

## Your job

Convert an existing slide deck (HTML, PowerPoint notes, or raw content) into the Strength Hack HTML presentation format. The output is a single self-contained `index.html` file that matches the design system exactly.

## Reference files in this project

| File | What it is |
|---|---|
| `HANDOVER.md` | **Read this first and in full.** Complete technical reference: design system, slide architecture, layout patterns, content rules, PDF export, common bugs. |
| `extracted/handoff/SKILL.md` | Original design system spec — fonts, type scale, Milo image rules |
| `extracted/handoff/colors_and_type.css` | All CSS variables and semantic classes |
| `extracted/handoff/example-slide.html` | One working reference slide |
| `index.html` | The live deck — use as the base template and pattern source |
| `assets/` | All brand images (Milo variants, BuchCover, app screenshots, logo) |

**Always read `HANDOVER.md` before writing a single line of HTML.**

## How to approach a conversion

### 1. Read the source material
Read the existing slide deck top to bottom. List every slide: title, body content, statistics, source citations. Note which slides are "extended" (only shown in 2h mode).

### 2. Ask before you build
Before writing HTML, confirm with the user:
- Which slides map to which chapter (`data-ch` 1–6)?
- Which slides are extended / 2h-only (`data-new="1"`)?
- Are there any slides that need new content written, or is it a straight conversion?

### 3. Map every slide to a layout pattern
Each slide must use one of the patterns documented in `HANDOVER.md` section 4:
- **Hero split** (grid-2) — headline + Milo image
- **Feature cards** (3-column grid) — module / use-case comparison
- **Stack** (vertical cards) — step-by-step / week list
- **Quote** (centred) — pulled quote
- **ROI grid** (2×2) — stats / proof points
- **Roadmap** (horizontal steps) — process / next steps
- **Section divider** (centred title + Milo) — chapter transition

### 4. Write slides using the exact HTML template
From `HANDOVER.md` section 3 — every slide follows this structure:
```html
<div class="slide dark" data-ch="N">
  <div class="frame between">
    <div class="top-bar">
      <div class="eyebrow">Chapter Title</div>
      <div class="page-num">N / 26</div>
    </div>
    <!-- content here -->
  </div>
  <div class="bottom-bar">   <!-- ← MUST be outside .frame -->
    <div>Left text</div>
    <div class="source-line">Right text</div>
  </div>
</div>
```

### 5. After writing, self-check every slide
- [ ] `bottom-bar` is outside `.frame` (sibling, not child)
- [ ] Every font-size references a `--type-*` variable (nothing below 24px)
- [ ] Headlines use DM Serif Display (`.display` / `.h2` classes)
- [ ] Eyebrow is uppercase, orange, above the headline
- [ ] Content fits in ~810px usable height (see `HANDOVER.md` section 7 for how to fix clipping)
- [ ] Terminology matches exactly: 6 Kompass dimensions, Apollo 13 role names, spectrum pairs (see `HANDOVER.md` section 8)
- [ ] Milo images: one per slide max, never text overlaid on Milo, always `.hero-image` wrapper

### 6. Update the JS at the bottom of the file
- Set `NEW_SLIDE_INDICES` to the correct 0-based indices of extended slides
- Set `CH_START` to the correct 0-based start index of each chapter

### 7. Test PDF export
Instructions in `HANDOVER.md` section 10. Check: all slides present, no sidebar, dark backgrounds visible, no image frame artefact.

## Hard rules

- **Never invent a third font.** DM Serif Display + Outfit only.
- **Never put `bottom-bar` inside `.frame`.** This causes overlap bugs.
- **Never use a raw px font-size** unless overriding a specific slide (document why).
- **Nothing below 24px** — the floor for projection legibility.
- **Milo faces toward the text.** Flip the column order if needed — never mirror the image.
- **`flex:1; min-height:0`** on any content area that needs to fill remaining height.
- **`align-self:stretch`** on `.hero-image` when it should fill grid cell height — but remove `height:auto` if present (they conflict).

## When content is too tall and clips

The frame has `overflow:hidden` — if content is clipped, it is genuinely too tall. Fix by:
1. Reducing the `h2` to 56–64px with an inline style
2. Reducing card padding (e.g. `28px 36px` → `20px 28px`)
3. Shortening or removing body text in cards
4. Removing a `<p class="sub">` paragraph
Do not increase the frame height or remove `overflow:hidden` — that breaks the bottom-bar.
