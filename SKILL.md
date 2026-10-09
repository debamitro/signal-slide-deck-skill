# Skill: HTML Slide Deck Builder

Create professional, presentation-grade slide decks as a single self-contained HTML file — no frameworks, no build tools, no external assets beyond Google Fonts. The deck runs in the browser with keyboard/touch navigation and prints to PDF on letter-size paper.

---

## Architecture Overview

A slide deck is one HTML file with three layers:

1. **Viewport + Stage** — a fixed-size design canvas (1920×1080) that auto-scales to fit any browser window
2. **Slides** — absolutely positioned full-bleed pages, shown one at a time via JS toggling `.active`/`.visible`
3. **Print layer** — `@media print` rules that reformat each slide onto letter landscape pages for PDF export

```
.deck-viewport (fixed, fills window)
  └─ .deck-stage (1920×1080, scaled to fit)
       └─ section.slide (one per page, absolutely positioned)
            └─ .slide-content (flex column, holds all visible content)
```

---

## Canvas & Scaling System

The design canvas is **1920×1080px** (16:9). All content is designed at this resolution. A JS `setupStageScale()` method calculates the scale factor on every resize and applies a CSS `transform` to `.deck-stage` to fit the viewport while preserving aspect ratio:

```js
setupStageScale() {
    const scale = () => {
        const factor = Math.min(window.innerWidth / 1920, window.innerHeight / 1080);
        const x = (window.innerWidth - 1920 * factor) / 2;
        const y = (window.innerHeight - 1080 * factor) / 2;
        this.stage.style.transform = `translate(${x}px, ${y}px) scale(${factor})`;
    };
    scale();
    window.addEventListener('resize', scale);
}
```

CSS for the stage:
```css
.deck-viewport { position: fixed; inset: 0; overflow: hidden; background: var(--stage-bg); }
.deck-stage { position: absolute; left: 0; top: 0; width: 1920px; height: 1080px; overflow: hidden; transform-origin: 0 0; background: var(--c-navy); }
```

---

## Color System

Use CSS custom properties for a cohesive two-tone system:

| Variable | Purpose |
|---|---|
| `--c-navy` | Dark slide background (`#1C2644`) |
| `--c-cream` | Light slide background (`#F0ECE3`) |
| `--c-gold` | Accent / emphasis / borders (`#C8A870`) |
| `--c-border-dark` | Borders on dark slides (`#2E3D5C`) |
| `--c-border-light` | Borders on light slides (`#CAC4B4`) |
| `--c-text-warm` | Primary text on dark bg (`#E2DCD0`) |
| `--c-ink` | Primary text on light bg (`#1A2030`) |
| `--c-text-muted-dark` | Secondary text on dark bg (`#8A96A8`) |
| `--c-text-muted-light` | Secondary text on light bg (`#5A6270`) |

**Light slides** use the `.light` class on `<section>`. Every text/border element must have a `.slide.light` variant that swaps to the appropriate color.

---

## Typography System

### Fonts (loaded via Google Fonts)
- **Source Serif 4** — headings (h1, h2, h3, stat values)
- **DM Sans** — body text, lead paragraphs, list items, labels
- **IBM Plex Mono** — kickers, chrome labels, counters, stat notes, controls

### Type Scale (as CSS custom properties)
| Variable | Size | Usage |
|---|---|---|
| `--font-display` | 182px | Hero stat numbers |
| `--font-h1` | 100px | Title slide heading |
| `--font-h2` | 58px | Section heading |
| `--font-h3` | 36px | Card heading |
| `--font-lead` | 27px | Lead/intro paragraph |
| `--font-body` | 20px | Body text, list items |

### Spacing Scale
| Variable | Size | Usage |
|---|---|---|
| `--pad-x` | 144px | Slide horizontal padding |
| `--pad-y` | 59px | Slide vertical padding |
| `--gap-lg` | 43px | Large gap (between major sections) |
| `--gap-md` | 27px | Medium gap (between elements) |
| `--gap-sm` | 13px | Small gap (tight spacing) |

---

## Slide Structure

### Basic Slide (dark background)
```html
<section class="slide">
    <div class="slide-content" style="padding: var(--pad-y) var(--pad-x);">
        <div class="chrome-bar">
            <span class="chrome-label">Section Name</span>
            <span class="chrome-counter">01</span>
        </div>
        <h2 class="reveal">Heading</h2>
        <div class="rule-gold reveal"></div>
        <!-- content here -->
    </div>
</section>
```

### Title/Closing Slide (no chrome bar)
```html
<section class="slide active">
    <div class="slide-content" style="padding: var(--pad-y) var(--pad-x);">
        <div class="kicker reveal">Label</div>
        <div class="rule-gold reveal"></div>
        <h1 class="reveal" style="margin-top: var(--gap-lg);">Title</h1>
        <p class="lead reveal" style="max-width: 600px; margin-top: var(--gap-md);">Subtitle text</p>
    </div>
</section>
```

### Light Slide (cream background)
```html
<section class="slide light">
    <!-- same structure, .light swaps all colors -->
</section>
```

**Only the first slide** gets the `active` class in the HTML.

---

## Component Reference

### Chrome Bar
Top-of-slide label + counter. Every content slide (not title/closing) gets one.
```html
<div class="chrome-bar">
    <span class="chrome-label">Section Name</span>
    <span class="chrome-counter">01</span>
</div>
```

### Kicker
Small uppercase label for title/closing slides.
```html
<div class="kicker reveal">Series Seed</div>
```

### Gold Rule
Short decorative horizontal rule. Place after headings.
```html
<div class="rule-gold reveal"></div>
```

### Lead Paragraph
Large intro text below headings.
```html
<p class="lead reveal" style="margin-top: var(--gap-md);">Text here</p>
```

### Stat Card
Border-top card for displaying a metric.
```html
<div class="stat-card reveal">
    <div class="stat-value" style="font-size: var(--font-display);">16M+</div>
    <div class="stat-label">Label text</div>
    <div class="stat-note">Source note</div>
</div>
```

### Two-Column / Three-Column Layouts
```html
<div class="two-col" style="margin-top: var(--gap-lg);">
    <div><!-- column 1 --></div>
    <div><!-- column 2 --></div>
</div>

<div class="three-col" style="margin-top: var(--gap-lg);">
    <div><!-- column 1 --></div>
    <div><!-- column 2 --></div>
    <div><!-- column 3 --></div>
</div>
```

### Bullet Lists
Uses em-dash bullets via `li::before`. No `<ul>` list-style.
```html
<ul>
    <li class="reveal">Plain point</li>
    <li class="reveal">Point with <em>emphasis</em></li>
</ul>
```

### Emphasis / Italic
Use `<em>` for inline emphasis. It renders in the gold accent color with italic style.
```html
<em>emphasized text</em>
```

---

## Reveal Animation System

Elements with the `.reveal` class start invisible (`opacity: 0; transform: translateY(30px)`) and animate in when their parent `.slide` gets the `.visible` class. Staggered delays are applied via `:nth-child`:

```css
.reveal { opacity: 0; transform: translateY(30px); transition: opacity 0.6s cubic-bezier(0.16, 1, 0.3, 1), transform 0.6s cubic-bezier(0.16, 1, 0.3, 1); }
.slide.visible .reveal { opacity: 1; transform: translateY(0); }
.reveal:nth-child(1) { transition-delay: 0.1s; }
.reveal:nth-child(2) { transition-delay: 0.2s; }
/* ... up to nth-child(6) at 0.6s */
```

**Apply `.reveal` to** every content element that should animate in: headings, paragraphs, list items, stat cards, rules, kickers. Do NOT apply it to structural containers (`.chrome-bar`, `.two-col`, `.three-col`).

---

## Navigation (JavaScript)

```js
class SlidePresentation {
    constructor() {
        this.slides = document.querySelectorAll('.slide');
        this.currentSlide = 0;
        this.stage = document.getElementById('deckStage');
        this.setupStageScale();
        this.setupKeyboardNav();
        this.setupTouchNav();
        this.setupControls();
        this.showSlide(0);
        this.updateIndicator();
    }
    setupStageScale() { /* ... see Canvas & Scaling System above ... */ }
    setupKeyboardNav() {
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowRight' || e.key === ' ') { e.preventDefault(); this.nextSlide(); }
            else if (e.key === 'ArrowLeft') { e.preventDefault(); this.prevSlide(); }
            else if (e.key === 'Home') { e.preventDefault(); this.showSlide(0); }
            else if (e.key === 'End') { e.preventDefault(); this.showSlide(this.slides.length - 1); }
        });
    }
    setupTouchNav() {
        let touchStartX = 0;
        document.addEventListener('touchstart', (e) => { touchStartX = e.touches[0].clientX; }, { passive: true });
        document.addEventListener('touchend', (e) => {
            const diffX = e.changedTouches[0].clientX - touchStartX;
            if (Math.abs(diffX) > 50) { diffX > 0 ? this.prevSlide() : this.nextSlide(); }
        }, { passive: true });
    }
    setupControls() {
        document.getElementById('prevBtn').addEventListener('click', () => this.prevSlide());
        document.getElementById('nextBtn').addEventListener('click', () => this.nextSlide());
    }
    updateIndicator() {
        document.getElementById('slideIndicator').textContent = `${this.currentSlide + 1} / ${this.slides.length}`;
    }
    showSlide(index) {
        this.currentSlide = Math.max(0, Math.min(index, this.slides.length - 1));
        this.slides.forEach((slide, i) => {
            slide.classList.toggle('active', i === this.currentSlide);
            slide.classList.toggle('visible', i === this.currentSlide);
        });
        this.updateIndicator();
    }
    nextSlide() { this.showSlide(this.currentSlide + 1); }
    prevSlide() { this.showSlide(this.currentSlide - 1); }
}
new SlidePresentation();
```

### Controls HTML
```html
<div class="deck-controls">
    <button class="control-btn" id="prevBtn">← Prev</button>
    <span class="slide-indicator" id="slideIndicator">1 / N</span>
    <button class="control-btn" id="nextBtn">Next →</button>
</div>
```

---

## Print / PDF System

This is critical — without proper print rules, the PDF will be blank or misaligned.

### Letter Landscape Math
- Letter landscape at 96 DPI = **1056 × 816 px**
- Scale factor: 1056 ÷ 1920 = **0.55**
- Scaled content height: 1080 × 0.55 = **594px**
- Vertical center offset: (816 − 594) ÷ 2 = **111px**

### Required Print Rules
```css
@media print {
    @page { size: letter landscape; margin: 0; }
    * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
    html, body { width: 1056px; height: auto; overflow: visible; background: var(--c-navy); }
    .deck-viewport { position: static; overflow: visible; background: var(--c-navy); }
    .deck-stage { position: static; width: auto; height: auto; transform: none !important; background: none; }
    .slide { position: relative; inset: auto; display: block !important; visibility: visible !important; opacity: 1 !important; pointer-events: auto !important; width: 1056px; height: 816px; overflow: hidden; break-after: page; }
    .slide-content { transform: scale(0.55); transform-origin: 0 0; width: 1920px !important; height: 1080px !important; margin-top: 111px !important; }
    .reveal { opacity: 1 !important; transform: none !important; }
    .slide::before { display: none; }
    .slide:last-child { break-after: auto; }
    .deck-controls { display: none !important; }
}
```

### Print Pitfalls (MUST follow)

| Pitfall | Cause | Fix |
|---|---|---|
| **Invisible text on dark slides** | Browsers strip background colors by default | `print-color-adjust: exact` on `*` + set `background: var(--c-navy)` on body/viewport (NOT `#fff`) |
| **Content missing in PDF** | `.reveal` elements have `opacity: 0` and JS never runs in print | `.reveal { opacity: 1 !important; transform: none !important; }` in print block |
| **Slides overflow page** | Raw 1920×1080 too large for letter paper | Scale `.slide-content` by 0.55, set `.slide` to 1056×816 |
| **Content not centered vertically** | Scaled content (594px) shorter than page (816px) | `margin-top: 111px` on `.slide-content` |
| **Grid overlay prints** | `.slide::before` background pattern renders | `.slide::before { display: none; }` in print block |

---

## Emphasis / Italic Spacing

Italic text slants right, causing the last character's upper strokes to visually overlap the next word. **Always** add inline margin to `<em>`:

```css
em { font-style: italic; color: var(--c-gold); margin-inline: 0.12em; }
```

The `margin-inline` adds breathing room on both sides. Adjust the value (0.08em–0.2em) based on the font's italic angle.

---

## Subtle Background Grid

Dark slides get a faint grid overlay via `.slide::before`:

```css
.slide::before { content: ""; position: absolute; inset: 0; background: linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px); background-size: 80px 80px; pointer-events: none; z-index: 0; }
.slide.light::before { display: none; }
```

This is hidden in print via `.slide::before { display: none; }`.

---

## Accessibility & Motion

Always include a reduced-motion media query:

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.2s !important;
    }
}
```

---

## Checklist Before Shipping

- [ ] All slides have `.slide-content` wrapper with padding
- [ ] Only first slide has `active` class
- [ ] Every content element has `.reveal` class (except structural containers)
- [ ] All `.slide.light` color variants defined for text/borders/cards
- [ ] `<em>` has `margin-inline` to prevent italic crowding
- [ ] `@media print` block includes all five critical fixes from the Pitfalls table
- [ ] `print-color-adjust: exact` present
- [ ] `.reveal { opacity: 1 !important }` present in print block
- [ ] Google Fonts `<link>` tags present in `<head>`
- [ ] Reduced-motion media query present
- [ ] Slide counter in `.chrome-counter` is two-digit zero-padded (01, 02, …)
- [ ] `slideIndicator` initial text matches total slide count
- [ ] Test print-to-PDF: all slides render, all text visible, fits letter landscape
