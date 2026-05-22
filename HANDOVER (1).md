# The Strength Hack — Slide Deck Handover

Complete technical reference for building, editing, and exporting the HTML slide deck in `index.html`. Read this before touching the file.

---

## 1. What the file is

`index.html` is a **self-contained 26-slide HTML presentation** for The Strength Hack leadership programme. It runs directly in any modern browser — no server, no build step, no dependencies except Google Fonts (loaded from CDN).

- Canvas: **1920 × 1080 px fixed**
- Browser scales the canvas with CSS `transform: scale()` to fit the viewport
- Sidebar (210 px) + stage area fill 100 vw / 100 vh
- All 26 slides exist in the DOM simultaneously; JS shows the active one (`opacity: 1`) and hides the rest (`opacity: 0`)

---

## 2. Design system

### Fonts — two only, never a third

| Family | Role | Weights |
|---|---|---|
| **DM Serif Display** | Every headline, large number, step number, quote | 400 (regular + italic) |
| **Outfit** | Everything else — body, eyebrows, labels, buttons, page nums | 300 400 500 600 700 |

Load via Google Fonts (already in `index.html` `<head>`):
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=Outfit:wght@300;400;500;600;700&display=swap" rel="stylesheet" />
```

CSS variables (defined in `:root`):
```css
--font-dm-serif: 'DM Serif Display', Georgia, serif;
--font-outfit:   'Outfit', system-ui, sans-serif;
```

### Colors

```css
--charcoal:    #1A1714;   /* slide background (dark slides) */
--cream:       #F5F0E8;   /* slide background (light slides) */
--ink:         #2C2824;   /* text on light */
--orange:      #D4652A;   /* brand accent — eyebrows, accents, highlights */
--on-dark:     #F5F0E8;   /* text on dark slides */
--on-dark-2:   rgba(245,240,232,0.62);  /* secondary text on dark */
--on-dark-3:   rgba(245,240,232,0.38);  /* muted text on dark */

/* Card variants */
--orange-soft: rgba(212,101,42,0.12);
--orange-line: rgba(212,101,42,0.30);
```

### Type scale (1920 × 1080 slides)

```css
--type-display: 132px;   /* hero title, slide 1 only */
--type-title:    88px;   /* big section title / divider */
--type-h2:       72px;   /* standard slide headline */
--type-quote:    56px;   /* pulled quotes */
--type-stat:    220px;   /* giant stat numbers */
--type-sub:      40px;   /* hero subhead, lede */
--type-body:     30px;   /* slide body copy */
--type-small:    24px;   /* footnotes, source lines, page numbers */
--type-eyebrow:  24px;   /* orange uppercase label above titles */
```

**Floor rule: nothing below 24 px.** Anything smaller fails projection legibility.

### Spacing scale

```css
--pad-x:       120px;  /* left + right padding inside every slide */
--pad-top:     110px;  /* top padding */
--pad-bottom:   90px;  /* bottom padding (protected zone for bottom-bar) */
--gap-title:    44px;  /* eyebrow → title → subtitle gap */
--gap-item:     28px;  /* list / grid item gap */
```

### Typography rules

- **Eyebrow**: Outfit 600, `text-transform: uppercase`, `letter-spacing: 4px`, color `var(--orange)`. Always **above** a headline, never below.
- **Italic accent**: `<em class="accent">` on one clause inside a headline only. DM Serif italic, orange.
- **Tracking**: `letter-spacing: -3px` on `--type-display`, `-1.8px` on `--type-title`, `-1.2px` on `--type-h2`.
- **Body**: `line-height: 1.4–1.5`, `text-wrap: pretty`, `max-width ≈ 720px`.

Semantic classes (use these, don't re-style):
```html
<div class="eyebrow">TEIL 1 · DAS PROBLEM</div>
<h1 class="display">Vom Testergebnis zur <em class="accent">Team-Excellence</em>.</h1>
<h2 class="h2">Stärken <em class="accent">skalieren</em>.</h2>
<p class="sub">Lede / subhead sentence.</p>
<p class="body">Body copy text.</p>
<span class="small">Footnote · source line</span>
```

---

## 3. Slide architecture

### Fixed canvas + JS scaling

Each slide is `position: absolute; width: 1920px; height: 1080px`. JavaScript scales the whole canvas to fill the stage:

```javascript
function scaleSlides() {
  const stageW = window.innerWidth - 210;  // 210 = sidebar width
  const stageH = window.innerHeight;
  const scale  = Math.min(stageW / 1920, stageH / 1080);
  const offX   = (stageW - 1920 * scale) / 2;
  const offY   = (stageH - 1080 * scale) / 2;
  SLIDES.forEach(s => {
    s.style.transform      = `scale(${scale})`;
    s.style.transformOrigin = 'top left';
    s.style.left           = offX + 'px';
    s.style.top            = offY + 'px';
  });
}
```

### HTML structure of a slide

```html
<div class="slide dark" data-ch="N">

  <div class="frame between">
    <div class="top-bar">
      <div class="eyebrow">Chapter Title</div>
      <div class="page-num">7 / 26</div>
    </div>

    <!-- slide content goes here -->

  </div>

  <!-- ⚠️ bottom-bar MUST be OUTSIDE .frame, as a sibling -->
  <div class="bottom-bar">
    <div>Left text</div>
    <div class="source-line">Right text · source</div>
  </div>

</div>
```

**Critical rule**: `.bottom-bar` is a **direct child of `.slide`**, not inside `.frame`. If you put it inside `.frame`, it will be overlapped by slide content. See section 6 for why.

### `.frame` CSS

```css
.frame {
  position: absolute;
  top: 0; left: 0; right: 0;
  bottom: var(--pad-bottom);  /* stops 90px above slide bottom = bottom-bar zone */
  padding: var(--pad-top) var(--pad-x) 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;            /* hard-clips content — prevents overlap with bottom-bar */
}
.frame.between { justify-content: space-between; }
.frame.center  { justify-content: center; align-items: center; }
```

### `.bottom-bar` CSS

```css
.bottom-bar {
  position: absolute;
  bottom: 20px;
  left: var(--pad-x);
  right: var(--pad-x);
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-family: var(--font-outfit);
  font-size: var(--type-small);
  color: var(--on-dark-3);
}
```

### Slide variants

- `class="slide dark"` — dark charcoal background (most slides)
- `class="slide"` — light cream background
- `data-ch="N"` — chapter number 1–6 (drives sidebar highlight)
- `data-new="1"` — marks slides only shown in 2h mode (hidden in 1h mode)

---

## 4. Layout patterns

### Two-column grid (most common)

```html
<div class="grid-2" style="gap:80px; align-items:center; flex:1; min-height:0;">
  <div> <!-- text column --> </div>
  <div class="hero-image"> <!-- image column --> </div>
</div>
```

Variants: `.grid-2-58` (58/42 split), `.grid-2-42` (42/58 split).

### Three-column feature cards

```html
<div style="display:grid; grid-template-columns:repeat(3,1fr); gap:28px; flex:1; min-height:0; align-items:stretch;">
  <div class="feat-card">
    <div class="eyebrow" style="margin-bottom:8px;">Modul 1</div>
    <div class="feat-title">Title</div>
    <div class="feat-body">Body copy.</div>
  </div>
  <div class="feat-card featured"> <!-- orange highlight variant --> </div>
  <div class="feat-card"> ... </div>
</div>
```

### Stack (vertical list of cards)

```html
<div class="stack" style="gap:14px;">
  <div class="card-dark" style="padding:20px 28px; display:flex; align-items:center; gap:20px;">
    <div style="font-family:var(--font-dm-serif); font-size:44px; color:var(--orange); line-height:1; flex-shrink:0;">W1</div>
    <div class="body">Card content</div>
  </div>
</div>
```

Use `card-dark` on dark slides, `card-orange` for the highlighted/CTA card.

### ROI tiles (2×2 grid)

```html
<div class="roi-grid" style="flex:1; min-height:0; align-items:stretch;">
  <div class="roi-tile">
    <div class="roi-num">–72%</div>
    <div class="roi-label">Fluktuation</div>
    <div class="roi-desc">Description text.</div>
  </div>
  <div class="roi-tile" style="background:var(--orange-soft); border-color:var(--orange-line);"> <!-- featured --> </div>
</div>
```

ROI tile CSS key sizes: `roi-num` = 72px DM Serif orange, `roi-label` = 44px, `roi-desc` = `--type-body`.

### Roadmap (horizontal steps)

```html
<div class="roadmap">
  <div class="roadmap-step">
    <div class="roadmap-week">Schritt 1</div>
    <div class="roadmap-title">Kickoff</div>
    <div class="roadmap-body">Short description.</div>
  </div>
</div>
```

### Milo hero image frame

All three ingredients are required:
1. `border-radius: 28px`
2. `background: #20322F` (teal backplate — shows when image has transparent edges)
3. `box-shadow: 0 20px 60px rgba(0,0,0,0.4)` (heavy float — heavier than UI cards intentionally)

```html
<div class="hero-image">
  <img src="assets/milo-hero.jpg" alt="Milo juggling crystals">
</div>
```

```css
.hero-image {
  align-self: stretch;          /* fills grid cell height */
  width: 100%;
  border-radius: 28px;
  overflow: hidden;
  background: #20322F;
  box-shadow: 0 20px 60px rgba(0,0,0,0.4);
}
.hero-image img { width:100%; height:100%; object-fit:cover; display:block; }
```

For fixed aspect ratio (e.g. on section divider slides):
```html
<div class="hero-image" style="aspect-ratio:4/5; height:auto;">
```
Do **not** combine `aspect-ratio` with `align-self:stretch` — they conflict. Choose one.

Book cover image uses `object-fit:contain` with matching background:
```html
<div class="hero-image" style="display:flex; align-items:center; justify-content:center; background:#1a2e2a;">
  <img src="assets/BuchCover.jpg" style="width:auto; height:100%; max-height:100%; object-fit:contain;">
</div>
```

### `flex:1; min-height:0` pattern

Any content area that should grow to fill remaining space after the top-bar needs both:
```html
<div style="flex:1; min-height:0;">
```
Without `min-height:0`, flex children can overflow their container even when `flex:1` is set.

---

## 5. Assets

All assets live in `assets/`. Fetch new files from `origin/main` when needed:
```bash
git fetch origin main
git checkout origin/main -- assets/
```

| File | Use |
|---|---|
| `milo-hero.jpg` | Slide 1 title, final CTA |
| `milo-strengths.jpg` | Strengths / individual focus slides |
| `milo-teamwork.jpg` | Team / collaboration slides |
| `milo-growth.jpg` | Roadmap / outcome slides |
| `BuchCover.jpg` | Book cover — use with `object-fit:contain` |
| `Noa1.jpg`, `Noa2.jpg` | Person photos |
| `all_assessment.jpg` | App screenshot — full dashboard |
| `coach_user.jpg` | App screenshot — AI coach UI |
| `first_assessment.jpg` | App screenshot — onboarding |
| `report_explorer.jpg` | App screenshot — team report |
| `strengths_explorer.jpg` | App screenshot — strength explorer |
| `team_creator.jpg` | App screenshot — team builder |
| `logo-mark-green.png` | Brand compass-arrow mark |

Milo rules:
- One Milo per slide maximum
- Never overlay text on Milo — headline goes beside the frame
- Always faces toward the text (flip column order if needed, never mirror image)
- Never add a colour gradient over Milo

---

## 6. Critical bug: bottom-bar overlap

### The problem

Early versions had `.bottom-bar` inside `.frame`. Because `.frame` is a flex column and slide content can grow tall, the bottom-bar gets pushed down and painted over by content — or vice versa.

### The fix (permanent)

1. **`.frame` ends before the bottom zone**: `bottom: var(--pad-bottom)` + `overflow: hidden`
2. **`.bottom-bar` is a sibling of `.frame`**, not a child — it's `position: absolute` relative to `.slide`

This means when you add a new slide, the bottom-bar must always be written **outside** `.frame`:

```html
<div class="slide dark" data-ch="N">
  <div class="frame between">
    ...content...
  </div>
  <div class="bottom-bar">   <!-- ← sibling of frame, NOT inside it -->
    <div>Left</div>
    <div class="source-line">Right</div>
  </div>
</div>
```

---

## 7. Content sizing — avoid clipping

The usable height inside `.frame` is approximately:
- 1080px total − 90px (pad-bottom) − 110px (pad-top) − ~70px (top-bar) = **~810 px**

At `--type-h2: 72px` a two-line headline takes ~150px. A 4-card stack at 70px each takes 280px + gaps. It adds up quickly.

**Common fixes when content gets clipped:**
- Reduce `h2` to 56–64px with `style="font-size:56px;"`
- Reduce card padding from `28px 36px` to `20px 28px`
- Remove or shorten body text in cards (`.sub` → `.body`, or delete entirely)
- Switch from 4 items to 3 items in a grid
- Use `style="font-size:96px"` on the slide-1 display headline (132px is too tall with a subhead)

---

## 8. Domain terminology (exact terms to use)

### Kompass — 6 Dimensions (always exactly 6)
1. Energie
2. Ausrichtung
3. Perspektive
4. Aktion ★ (orange highlight card)
5. Struktur
6. Sicherheit

### Apollo 13 Roles (always these 5)
1. Kommandant
2. Pilot
3. Ingenieur
4. Kommunikation
5. Krisenstab

### Kompass Spectrum Pairs (use these exact terms)
- Impulsiv ↔ Bedacht
- Organisiert ↔ Agil

### CliftonStrengths Domains
- Beziehungsaufbau (Relationship Building) — blue
- Beeinflussung (Influencing) — yellow
- Ausführung (Executing) — red
- Strategisches Denken (Strategic Thinking) — green

### App modules
- **Modul 1**: Team-Stärken-Karte
- **Modul 2**: AI-Coach
- **Modul 3**: Wöchentliches Momentum (Präsentationsmodus für Workshop-Meetings)

### Content guidelines (slides 22–26)
- Avoid "Baseline setzen" and "Re-Score" language — not every engagement has a clear baseline
- Frame the journey as **Storming → Performing**, not sprint start → end measurement
- ROI tiles: qualitative benefits first (Weniger Reibung, Mehr Motivation), Gallup stats second
- Slide 26 CTA: direct to strength-hack.com, not a "Discovery Call" CTA
- Use Case 3 (slide 24): Team-Entwicklung = Blinde Flecken, Gaps, Reibungen, Rollenklärung

---

## 9. Navigation system

### Sidebar

- 6 chapter buttons (`data-ch="0"` to `data-ch="5"`)
- Dot grid (one dot per slide, numbered, click to jump)
- Mode button: toggles **2h Modus** ↔ **1h Modus**
- Export PDF button (see section 10)

### 1h / 2h mode

Slides with `data-new="1"` are "extended" slides (only in 2h version). In 1h mode they are hidden and skipped during navigation.

```javascript
const NEW_SLIDE_INDICES = new Set([5, 6, 10, 11, 12, 13, 15, 16, 17, 18, 19]);
```

When adding a new extended slide, add its 0-based index to this Set.

### Keyboard shortcuts
- `→` / `Space` / `↓` — next slide
- `←` / `↑` — previous slide
- `Home` / `End` — first / last slide
- `M` — toggle 1h/2h mode

---

## 10. PDF export

### How it works

An `@media print` CSS block resets the slide system for printing:
- Hides sidebar, arrows, counter, export button
- Makes `.app` `display:block` (removes flex layout)
- Sets `.stage` `margin-left:0` (removes sidebar offset)
- Makes `.slides-wrap` `position:static` so slides flow vertically
- Makes each `.slide` `position:relative; display:block` — they stack, one per printed page
- Forces exact background/color rendering (`-webkit-print-color-adjust: exact`)
- Removes `.hero-image` box-shadow (looks like a border artefact on paper)
- `@page { size: 1920px 1080px; margin: 0; }` — each page exactly matches the slide canvas

### Using it in Chrome

1. Click **"↓ Export PDF"** in the sidebar
2. Chrome print dialog opens
3. Set **Destination** → "Als PDF speichern"
4. Set **Ränder** → "Keine" (No margins)
5. Tick **"Hintergrundgrafiken"** (Background graphics) — required for dark slides
6. Click **Speichern** → single PDF, all 26 slides, one per page

### 1h mode awareness

If 1h mode is active when you click Export PDF, the JS temporarily marks extended slides with `data-skip-print`, which the print CSS hides. After printing the `afterprint` event cleans up the attributes.

### The exact CSS block (copy this if rebuilding)

```css
@media print {
  @page { size: 1920px 1080px; margin: 0; }

  * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }

  .sidebar, .arr, .counter, #exportBtn { display: none !important; }

  body, html { width: 1920px !important; height: auto !important; overflow: visible !important; }
  .app { display: block !important; width: 1920px !important; height: auto !important; }

  .stage {
    display: block !important;
    margin-left: 0 !important;
    overflow: visible !important;
    width: 1920px !important;
    height: auto !important;
    background: transparent !important;
  }

  .slides-wrap {
    position: static !important;
    width: 1920px !important;
    height: auto !important;
    overflow: visible !important;
  }

  .slide {
    position: relative !important;
    transform: none !important;
    left: 0 !important; top: 0 !important;
    opacity: 1 !important;
    pointer-events: auto !important;
    width: 1920px !important; height: 1080px !important;
    page-break-after: always; break-after: page;
    overflow: hidden !important;
    display: block !important;
  }

  .slide[data-skip-print] { display: none !important; }
  .hero-image { box-shadow: none !important; }
}
```

### The export button HTML

```html
<button class="mode-btn" id="exportBtn" style="margin-top:8px; opacity:0.7;" onclick="exportPDF()">
  ↓ Export PDF
</button>
```

### The export JS function

```javascript
function exportPDF() {
  if (skipNew) {
    SLIDES.forEach((s, i) => {
      if (NEW_SLIDE_INDICES.has(i)) s.setAttribute('data-skip-print', '1');
    });
  }
  window.print();
  window.addEventListener('afterprint', () => {
    SLIDES.forEach(s => s.removeAttribute('data-skip-print'));
  }, { once: true });
}
```

### Common export problems

| Symptom | Cause | Fix |
|---|---|---|
| Sidebar still visible | Used `#sidebar` (ID) but element has `class="sidebar"` | Use `.sidebar` selector |
| Only 1 slide in PDF | `.stage` still has `margin-left:210px`; `.app` still flex | Set `.app { display:block }` and `.stage { margin-left:0 }` |
| Dark slides print white/grey | "Hintergrundgrafiken" not ticked | Tick it in Chrome print dialog |
| Image has dark frame border | `hero-image` box-shadow prints as visible border | `@media print { .hero-image { box-shadow: none !important; } }` |
| Content cut off on right | Page width wrong | Ensure `@page { size: 1920px 1080px }` and `.slide { width: 1920px }` |

---

## 11. Converting a new presentation into this format

Step-by-step process for bringing a new HTML/PowerPoint deck into the Strength Hack slide system:

### Step 1 — Inventory the source content
List all slides: title, body text, any statistics, source citations. Note which slides are "extended" (2h only).

### Step 2 — Map slides to layout patterns
Each slide should use one of these patterns:
- **Hero split** (grid-2): big headline + Milo image
- **Feature cards** (3-column grid): module / use-case comparison
- **Stack** (vertical cards): step-by-step / week-by-week list
- **Quote** (centred): pulled quote in `--type-quote` DM Serif italic
- **ROI grid** (2×2): statistics / proof points
- **Roadmap** (horizontal steps): next-steps / process
- **Section divider** (centred title): chapter transition, big Milo

### Step 3 — Set up the shell
Copy the existing `index.html` structure: `<aside class="sidebar">`, `<main class="stage">`, the slides-wrap div, and the full `<script>` block. Update chapter names in sidebar. Set `NEW_SLIDE_INDICES` to the correct indices.

### Step 4 — Write each slide
Follow the HTML template from section 3. Always:
- Open `<div class="slide dark" data-ch="N">`
- Add `<div class="frame between">` with `<div class="top-bar">` inside
- Write content inside frame using layout patterns from section 4
- Close frame, then add `<div class="bottom-bar">` **outside** frame as sibling
- Close slide div

### Step 5 — Check sizes
Open in Chrome and visually check every slide:
- No content clipped at the bottom? (frame has `overflow:hidden` — clipping means content is too tall)
- Text readable at 1920×1080? (min 24px, ideally 30px+ for body)
- Bottom-bar visible and not overlapped?

If content clips: reduce font sizes, card padding, or amount of copy (see section 7).

### Step 6 — Terminology pass
Run through slides 17–21 specifically and verify Kompass dimensions (6, not 4), Apollo 13 roles, and spectrum pairs match the exact terms in section 8.

### Step 7 — Test PDF export
Click Export PDF, check Chrome print preview shows all slides, no sidebar, dark backgrounds intact. See section 10.

### Step 8 — Commit and push
```bash
git add index.html assets/
git commit -m "Description of changes"
git push -u origin <branch-name>
```

---

## 12. Chapter map

| data-ch | Slides | Chapter title |
|---|---|---|
| 1 | 1–3 | Hook & Problem |
| 2 | 4–9 | Wissenschaft & Neurologie |
| 3 | 10–13 | Persönliche Inventur |
| 4 | 14–20 | Team-Dynamik & Kompass |
| 5 | 21–24 | App als Cockpit |
| 6 | 25–26 | Next Steps & ROI |

`CH_START = [0, 3, 10, 14, 21, 24]` — 0-based start index of each chapter (used in JS).
