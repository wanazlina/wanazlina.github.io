# Source fixes for the Astro project

This repository holds only the **built output** of the site. A few small fixes
were applied directly to the built HTML on branch `claude/site-improvements`.
The next Astro build will overwrite them, so each one is listed below with how
to port it to the source. Part 2 lists issues found in the audit that should be
fixed in the source (not applied here). This file is not linked from the site
and is disallowed in `robots.txt`.

Audit date: 5 Nov 2026. Checked: all 27 HTML pages (13 EN, 13 MS, 404) at
375 px, 768 px and 1280 px in headless Chromium, plus a static parse of every
page.

---

## Part 1. Changes applied to the built output

| # | File(s) | Change | Why | Port to source |
|---|---------|--------|-----|----------------|
| 1 | `404.html` | Removed `<link rel="canonical" href=".../404/">`, the three `hreflang` alternates (`/404/`, `/ms/404/`, x-default) and `og:url`. | Those URLs do not exist. `/ms/404/` was never built. The page is already `noindex`. | In the layout's head component, skip canonical, hreflang and `og:url` when the page is the 404 page (e.g. pass a `noindex`/`is404` prop). |
| 2 | `impact/index.html` | "1 papers" changed to "1 paper" (9 map `<title>`s, 9 `aria-label`s, 9 country panels). "1 citations" changed to "1 citation" (4 chart tooltips, 4 `aria-label`s). | Grammar, including in what screen readers announce. | Pluralise in the co-author map and citations-by-year chart components, e.g. `` `${n} ${n === 1 ? 'paper' : 'papers'}` ``. The MS text ("makalah", "petikan") needs no change. |
| 3 | `ms/impact/index.html` | "Scopus (Elsevier API), author ID 57203833103" changed to "... ID pengarang 57203833103" (4 places). | English left on the Malay page. | Translate the source-label string in the MS dictionary or impact data. |
| 4 | `moments/index.html`, `ms/moments/index.html` | Added `width`/`height` (the poster JPGs' real pixel sizes) to the 19 `/videos/clips/*.jpg` poster `<img>`s. | They were the only content images on the site without intrinsic size. | Add `width`/`height` to the reel poster `<img>` in the Moments page or component, or use `<Image>`. |
| 5 | same | Reel click handler: the new `<video>` now gets `aria-label` (taken from the button label without "Play clip:" / "Mainkan klip:"), and focus moves to it (`t.focus()`). | Before, focus fell back to `<body>` and the video had no accessible name. The News page player already does both. | Same two calls in the Moments page's inline `<script>`. |
| 6 | same | Added `<style>.reels{scroll-padding-inline:max(16px,calc((100vw - 1200px)/2))}</style>` before `</head>`. | `.reels` uses `scroll-snap-type:x mandatory` with inline padding but no `scroll-padding`, so the browser snapped the first phone flush to the screen edge (pre-scrolled 16 px at 375 px and 40 px at 1280 px). | Add `scroll-padding-inline` to the `.reels` rule in the Moments page styles. |
| 7 | `research/index.html`, `ms/research/index.html` | Added `<style>.slides{scroll-padding-inline:6px}</style>`. | Same snap bug on the slide carousels (pre-scrolled 6 px, so the first slide's focus ring was clipped). | Add `scroll-padding-inline: 6px` to `.slides` in the slides component. |
| 8 | `robots.txt` | Added `Disallow: /SOURCE-FIXES.md`. | Keeps this file out of search. | Not needed in the source unless this file is kept there; `robots.txt` lives in `public/`. |

No hashed `_astro/` file was edited, renamed or deleted.

---

## Part 2. Fix in the Astro source (not applied here)

Ordered by priority. **[verify]** means it needs the owner's knowledge, so it
was not changed.

### P1: content accuracy (real-person facts)

1. **[verify] Melbourne postdoc dates disagree.** Home career pathway: "Postdoctoral Fellow, University of Melbourne (2008–2009)". About: "spent 2008 to 2012 as a postdoctoral and visiting scholar". CV and PDF: "Jun 2008–Mar 2012 Postdoctoral/Visiting Scholar". Pick one wording and use it on all three (EN and MS).
2. **[verify] Academic Coordinator dates disagree.** Home: "Academic Coordinator (2012–2014)". CV and PDF: "2012–present Academic Coordinator".
3. **[verify] "Twenty-five years at UPM"** (home career pathway, EN and MS). She joined as a tutor in Sep 2000, so it is now 26 years. Consider "More than twenty-five years", or compute it from the start year.
4. **[verify] Name variant.** Group page, CCBU members list (EN and MS): "Wan Azlina Wan **Abdul** Karim Ghani". The rest of the site, the JSON-LD and the CV use "Wan **Ab** Karim Ghani".
5. **[verify] "c. 2021 Research sabbatical, AAIBE Chair for Renewable Energy, UNITEN"** (CV, PDF). "c." reads as uncertain on a CV. Give the exact dates if known.
6. **[verify] Typos in publication titles.** Check them against the publisher and fix if they are transcription errors:
   - "Porous carbon foams supported rGO-ppy//rGO for **asymmteric** supercapacitor device" (home, research, publications, EN and MS)
   - "... Bunch-Based Cellulose for **Levulini** Acid Production" (publications)
   - "biochar as potential adsorbent for heavy metal **removal removal** from wastewater" (publications)
7. **Messy author lists in the publications archive** (the non-Scopus records, which look imported from a university system). Examples: "Robiah, Y.H. Taufiq-Yap, U. Rashid , -; ...", "Wan Azlina, W. A.K.G., , -, -, -, -; razifharun", "THOMAS, , U. Rashid , azie", "haa sidek". Clean the data file, or drop empty and "-" entries when rendering.
8. **[verify] Course code.** Teaching lists "ECH 3116 Heat and Mass Transfer Processes" (2016/17–2019/20, and the syllabus resource) and "ECH 3125 Heat and Mass Transfer Processes, 2024/2025". Confirm the code changed rather than being a typo.

### P2: English / Malay sync

9. `ms/teaching/`: an untranslated English paragraph sits between Malay paragraphs: "This outcome-based framework forces students to step away from idealized textbook scenarios and physically construct what they have mathematically modeled." (The other English passages on that page are quoted testimonials and resource titles, which is fine.)
10. Otherwise the two locales are in good sync. Every page pair has the same structure and the same numbers. All MS internal links stay in `/ms/`. `lang`, `og:locale` and hreflang are correct everywhere.

### P3: accessibility

11. **Zoomable images** (Base layout script): every `img` in `<main>` that is not in a link or button gets `tabindex=0` but no `role="button"` or name such as "Enlarge photo". It also catches the `<img>` inside the open `dialog.lightbox`, so the lightbox image becomes a tab stop, and Enter on it opens a second "zoom" dialog on top. Add `dialog` to the exclusion selector (`t.closest("a, button, dialog, ...")`) and set `role="button"` plus an `aria-label`.
12. **Language toggle tap targets** (header, every page): "EN" and "BM" are 18–20 × 42 px. They probably pass WCAG 2.5.8 through the spacing exception, but horizontal padding to at least 24 px (ideally 44 px) wide makes them easier to hit on phones.
13. Footer and source-note links are 18–21 px tall at 375 px. They are acceptable as text links, but `padding-block: 4px` would help touch users.
14. Lightbox captions taken from Malay headlines on EN pages (e.g. News, MajalahSains item) are announced in English. Pass `lang` through `data-caption` if needed (low priority).
15. Checked and fine: skip link, visible focus rings (3 px), one `h1` per page, no skipped heading levels, labelled form controls (publications filters), lightbox keyboard support (Enter/Esc/arrows, 48 × 48 px buttons, focus returns to the opener), menu dialog, News video facades, `prefers-reduced-motion` (no running animations, reveal content visible), content visible with JS off, and no text below AA contrast found by a computed-style scan (text over photos was not machine-checked).

### P4: SEO

16. The `og:image` is `og.webp`. Some link-preview crawlers (notably LinkedIn and several messaging apps) do not render WebP. Add a 1200 × 630 JPG or PNG and point `og:image` at it. Also consider adding an explicit `twitter:image`.
17. EN and MS home pages share the same `<title>`. That is acceptable for a name, but a short locale suffix helps (e.g. "... | Profesor, UPM").
18. Some descriptions are long and will be truncated in search results: `/impact/` (177 chars), `/research/` (175), `/ms/research/` (194), `/ms/impact/` (163).
19. The 404 page reuses the home description. Harmless because it is `noindex`, but a short "Page not found" description reads better when shared.
20. No `apple-touch-icon` or PNG favicon (only `favicon.svg`), so iOS home-screen and some browsers fall back to a screenshot.
21. Checked and fine: unique titles and descriptions on every other page, self-canonicals, `en`/`ms`/`x-default` hreflang on all 26 content pages, Open Graph on all pages, valid `Person` JSON-LD, and a sitemap that lists exactly the 26 built pages.

### P5: performance and repo size

The repo is about 501 MB on disk: a 256 MB working tree plus a 245 MB `.git` pack.

22. **79 MB of unused original JPGs in `_astro/`.** 408 `*.jpg` files and 1 PNG are not referenced by any HTML, CSS or JS. Only their WebP derivatives are used. The largest: `a3abbf83f7.D7hIr57Z.jpg` (572 KB), `ig0483-2.c7VuGMF3.jpg` (530 KB), `ig0888-0.CxCewx93.jpg` (504 KB), `ig0978-14.DBb2E7R6.jpg` (484 KB). This usually means the source imports images and uses `image.src` somewhere (for example to build lightbox `href`s or `og:image`), which makes Astro emit the original. Use `getImage()` output for those URLs instead. Each rebuild re-adds these files.
23. **Videos: 53 MB in `/videos/`.** `kans-award-2018.mp4` is 16.3 MB (4:08, 854×480). The 19 reel clips total 36 MB, the largest being `ig1022.mp4` (8.3 MB, 2:03) and `fb0270.mp4` (5.0 MB). Bitrates are already low (about 0.2–0.5 Mbps), so recompressing would save little. The big win is to host the KANS video on YouTube, like the two other News videos, and keep only short reels locally. Recompressing in place would bloat the git history, so it was not done.
24. **WebP variants: 1,690 files, 115 MB.** The largest single files are about 380 KB (`ig0483-2…_1IDvHr.webp`, `ig0888-0…_1vt6w5.webp`). Consider capping the largest widths at 1400 px and quality at about 75.
25. **Page weight (1280 px, after scrolling the whole page):** Moments about 14.8 MB, News about 7.8 MB, Gallery about 6.5 MB. Initial load at 1280 px: home 1.5 MB, because the hero slideshow images, though marked `loading="lazy"`, sit in the viewport and so load at once. Load slides 2+ after `requestIdleCallback`, or render them only once the slideshow advances. On Moments, the multi-photo cards use `sizes="(max-width: 700px) 100vw, 40vw"` for photos that display at about 20vw in the 4-up grid. Tighten `sizes` per grid.
26. **Large HTML documents:** Gallery 502 KB, Moments 487 KB, Publications 483 KB, Impact 421 KB (uncompressed; GitHub Pages gzips them). Impact appears to render the citations-by-year chart twice, probably as separate mobile and desktop versions. Consider paginating the Gallery and Moments, or loading later years on demand.
27. **Page scripts emitted after `</html>`** on Gallery, Moments and News (EN and MS). Browsers tolerate it, but it is invalid HTML. Move those `<script>`s inside the layout's slot or `<body>`.
28. Checked and fine: every image except the 19 posters fixed above has `width`/`height` (CLS is at most 0.01 on every page), below-the-fold images are lazy, the LCP image uses `fetchpriority="high"`, fonts are self-hosted WOFF2 with `font-display: swap`, and each page loads 2–3 render-blocking stylesheets of about 40–80 KB in total.

### P6: minor polish

29. Footer shows both an "SPERC" badge and a text link to the same `https://sperc-upm.github.io`. One is enough.
30. Spelling is mixed: "program" and "programme", "modelled" and "modeled", "optimise" and "optimization" in site copy, whereas the copy mostly uses British spelling. Paper titles should stay as published.
31. Third-party visit counter (`abacus.jasoncameron.dev`) is called on Home and About. It fails silently and the badge stays hidden, which is good. Mention it in a privacy note if UPM policy requires one.

---

## Verification numbers (before → after this branch)

- Local broken links, anchors, images or assets: 0 → 0 (static parse of every `href`/`src`/`srcset`)
- Local failed requests or JS errors in Chromium (27 pages × 3 widths): 0 → 0. The only failures were third-party hosts blocked in the test sandbox (YouTube thumbnails, visit counter).
- Horizontal overflow at 375, 768 and 1280 px: 0 pages → 0 pages
- Content images without `width`/`height`: 38 → 0
- Canonical/hreflang tags pointing at non-existent URLs: 4 → 0
- "1 papers" / "1 citations": 35 → 0
- Scroll-snap strips that load pre-scrolled past their padding: 4 per locale (1 reels strip and 3 research carousels) → 0
- External links (325 unique, 135 of them DOIs) could not be checked from the sandbox. Run a link checker such as `lychee` from a normal network.
