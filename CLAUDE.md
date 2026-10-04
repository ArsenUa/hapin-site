# CLAUDE.md

Static marketing site for **hapin.co**, served by GitHub Pages straight from the repo root (`CNAME` → `hapin.co`). There is no build step, package manager, framework or JS bundle: plain HTML files, one shared stylesheet, and SVG assets. To preview, open a page in a browser or run `python3 -m http.server` from the repo root.

## Structure

```
index.html      Home: centered hero (icon, tagline, "Coming soon" status)
privacy.html    Privacy Policy: .doc layout with a <details> accordion
contact.html    Contact Us: .doc layout with .contact-list (inline SVG icons)
styles.css      The only stylesheet; shared by every page
assets/         Logos and icons (SVG only)
CNAME           GitHub Pages custom domain — don't edit
```

### Pages

Every page repeats the same shell (there are no includes or templates, so shared markup is copy-pasted):

1. `<head>`: charset, viewport, `<title>`, Google Fonts preconnects + Fredoka link, `styles.css`, favicon `assets/hapin_logo_white_letters.svg`.
2. `<header class="site-header">`: logo link to `index.html` (`assets/hapin_logo_white.svg`) and the `.menu-toggle` hamburger button.
3. `<nav id="nav-panel" class="nav-panel">`: full-screen overlay menu. The current page's link has `class="is-current"` (renders in the gradient).
4. `<main>`: page content.
5. `<footer class="site-footer">`: `&copy; 2026 Hapin, Inc. All rights reserved.`
6. Inline `<script>` at the end of `<body>` that toggles the nav panel (click + Escape) and keeps `aria-expanded` / `aria-label` in sync.

Title convention: `Page Name — Hapin` (em dash). The home page is `Hapin — Coming Soon`.

### styles.css

- Design tokens live in `:root` — always use the variables, never hard-code colors.
- Classes are BEM-ish: `block`, `block__element`, state classes `is-open` / `is-current`.
- Layout blocks:
  - `.hero` (+ `__mark`, `__tagline`, `.accent`, `__status`): the centered home-page hero.
  - `.doc` (+ `__eyebrow`, `__updated`, `h1`): the text-page column, max width `--max-width` (720px).
  - `.accordion` / `.accordion-item` / `.accordion-item__body` / `.plus`: collapsible sections built on `<details>/<summary>`.
  - `.contact-list` (+ `__icon`): bordered list rows with an icon.
  - `.draft-notice`: elevated callout box (defined but not currently used on any page).
- `main` is a centered flex container by default (for the hero). Text pages override it inline with `<main style="display:block; padding-top:24px;">`.
- Respects `prefers-reduced-motion` (transitions disabled); focus rings use `--focus`.

### assets/

| File | Use |
| --- | --- |
| `hapin_logo_white.svg` | Header wordmark (every page) |
| `hapin_logo_white_letters.svg` | Favicon (every page) |
| `hapin_icon.svg` | Gradient brand mark in the home hero |
| `icon-mail-open.svg`, `icon-phone.svg`, `icon-location-marker.svg` | Source for the contact icons. `contact.html` inlines these SVGs (with `stroke="currentColor"`) so they pick up `--text-muted`; the files themselves aren't referenced. |

Keep new assets as SVG where possible, named in lowercase with hyphens or underscores like the existing files.

## Brand

Source: Hapin Visual Style Guidelines V02 (June 2026). Dark theme only.

| Token | Value | Use |
| --- | --- | --- |
| `--bg` | `#111111` | Page background |
| `--bg-elevated` | `#1a1a1a` | Raised surfaces (e.g. `.draft-notice`) |
| `--text` | `#ffffff` | Primary text |
| `--text-muted` | `#a3a3a3` | Secondary text, body copy in accordions, icons, footer |
| `--hairline` | `#2a2a2a` | 1px dividers and borders |
| `--blue` | `#4a89ff` | Gradient start |
| `--green` | `#38fca3` | Gradient end; links, accordion `+`, focus ring |
| `--gradient` | `linear-gradient(90deg, #4A89FF → #38FCA3)` | Brand accent: gradient text (`.accent`, `.is-current`), status dot |

Gradient text is done with `background: var(--gradient); -webkit-background-clip: text; background-clip: text; color: transparent;`.

**Typeface: Fredoka** (Google Fonts, weights 400/500/600/700), with the fallback stack `'Fredoka', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` set on `body`. Headings use 600, nav links and accordion summaries 500, body 400. Every page must include the font link:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@400;500;600;700&display=swap" rel="stylesheet">
```

## Adding a new page

1. **Copy an existing page** as the starting point: `contact.html` for a short text page, `privacy.html` for long-form/accordion content, `index.html` only for a hero-style landing page. Name the file in lowercase, e.g. `about.html`.
2. **Update `<head>`**: set `<title>New Page — Hapin</title>`; optionally add a `<meta name="description">` (only `index.html` has one today). Leave the font links, `styles.css` link and favicon as they are.
3. **Add the nav link to every page.** Add an `<li><a href="new-page.html">New Page</a></li>` to `.nav-panel__list` in `index.html`, `privacy.html`, `contact.html` and the new page, in the same order everywhere. On the new page only, put `class="is-current"` on its own link and remove it from the others.
4. **Write the content** inside the standard text-page wrapper:

   ```html
   <main style="display:block; padding-top:24px;">
     <div class="doc">
       <p class="doc__eyebrow">Section label</p>
       <h1>New Page</h1>
       <p class="doc__updated">Subtitle or "Last updated: Month D, YYYY"</p>

       <!-- content: paragraphs, .accordion, .contact-list, .draft-notice … -->
     </div>
   </main>
   ```

   Accordion items follow this pattern (add `open` to the first one):

   ```html
   <div class="accordion">
     <details class="accordion-item" open>
       <summary>1. Heading <span class="plus">+</span></summary>
       <div class="accordion-item__body">
         <p>…</p>
       </div>
     </details>
   </div>
   ```

5. **Keep the header, footer and nav script identical** to the other pages; don't change the copy-pasted script per page.
6. **Styling**: reuse existing classes first. If something new is needed, add it to `styles.css` (not inline or in a `<style>` block), build it from the `:root` tokens, follow the BEM-ish naming, and check it at mobile width (existing styles use `clamp()` for responsive sizing rather than media queries).
7. Use relative links (`index.html`, `assets/...`) — the site is served from the domain root and there's no router.
