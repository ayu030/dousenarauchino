# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**どうせならうちのこ** is a Japanese static website that distributes free Canva T-shirt design templates for pets. Users open a Canva link, swap in a photo of their pet, and send the result to a print service.

No build system, no package manager, no framework. Just two self-contained HTML files served statically.

## Development

Open files directly in a browser — there is no build step, dev server, or toolchain.

```bash
# macOS
open index.html
open template-01.html

# Linux
xdg-open index.html
```

## File Structure

| File | Purpose |
|------|---------|
| `index.html` | Landing page: hero, template gallery, T-shirt picks, how-to, order CTA |
| `template-01.html` | Per-template detail page with photo gallery and Canva link |

Each file is fully self-contained — `<style>` block for CSS, inline `<script>` at the bottom for JS. There are no external JS files, no CSS files, and no shared partials.

## Architecture

### Design System (CSS Variables)

Both files define identical `:root` variables:

```css
--bg: #ffffff
--bg-sub: #f8f9fa
--accent: #0b57d0
--accent-hover: #0842a0
--text-main: #202124
--text-sub: #5f6368
--border: #e8eaed
```

When adding new pages or modifying colors, keep these variables in sync across all files manually.

### Template Card Pattern (index.html)

Each template card in the `templates-grid` pairs a photo carousel with dot indicators. The IDs must follow the `scroll-card{N}` / `dots-card{N}` naming convention, which the JS functions depend on:

```html
<div class="template-card" onclick="window.location.href='template-0N.html'">
  <div class="photo-scroll" id="scroll-card{N}">
    <div class="photo-scroll-item">...</div>
  </div>
  <div class="template-info">
    <div class="photo-dots" id="dots-card{N}">
      <button class="photo-dot active"
        onclick="event.stopPropagation();goToPhoto('scroll-card{N}','dots-card{N}',0)"></button>
    </div>
    <h3>デザイン0N</h3>
    <a href="template-0N.html" class="canva-link">詳しく見る</a>
  </div>
</div>
```

### Photo Carousel JS (index.html)

`goToPhoto(scrollId, dotsId, index)` — scrolls the carousel and syncs dot state.

`DOMContentLoaded` wires touch (swipe-through vs. tap-to-advance) and click handlers on every `.photo-scroll` element automatically; no per-card JS needed.

Pagination (`setPage`, `changePage`) is client-side state only — it does not filter or load content; it just tracks the current page number visually.

### Template Detail Pattern (template-01.html)

The `photos[]` array at the top of the `<script>` block drives the thumbnail gallery. Entries with a `src` key display an image; entries with `placeholder: true` display an emoji stand-in:

```js
const photos = [
  { src: 'design-01.jpg', label: 'Canvaデザイン' },
  { placeholder: true, emoji: '👕', label: '着用写真（準備中）' },
];
```

`switchPhoto(index)` reads this array to update the main photo and active thumbnail state.

## Adding a New Template

1. Copy `template-01.html` → `template-0N.html`
2. Update the `photos[]` array, title, description, and Canva link inside the new file
3. Add a card to the `templates-grid` in `index.html` following the card pattern above, incrementing the card number in all IDs

## Responsive Breakpoint

Both files use a single breakpoint at `max-width: 760px`. On mobile, the detail page (`template-01.html`) collapses from a 2-column grid to a single column and unsticks the photo panel.

## Language & Font

All user-facing copy is in Japanese. The font is **Zen Kaku Gothic New** loaded from Google Fonts. All weights used: 300, 400, 500, 700, 900.
