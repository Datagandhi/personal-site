# personal-site

Static portfolio site for Naveen Raj Mohan. Plain HTML/CSS/JS, no build
step, hosted on GitHub Pages at https://naveenrajmohan.com.

This README is the design + functionality contract for the site. When
adding anything new, **extend the patterns documented here rather than
introducing parallel ones**. If a new pattern is genuinely needed, add it
here in the same shape as the existing entries so the next change can
follow it.

---

## 1. Site map

| Path | Role |
|------|------|
| `/` (`index.html`) | Redirects to `/portfolio.html` via `<meta http-equiv="refresh">` + `window.location.replace`. About / Endorsements content is preserved in the source so the homepage can be re-enabled by deleting the three redirect lines in the `<head>`. |
| `/portfolio.html` | **Front door.** Power BI dashboards grid + "More work" links. |
| `/dbt-estate.html` | Project entries built with dbt. Currently: House Hunt System. |
| `/case-studies.html` | Long-form write-ups. Placeholder for now. |
| `/assets/` | Image assets. PNG screenshots of Power BI reports live here. |
| `/styles.css` | Single stylesheet for the whole site. |
| `/CNAME` | Custom-domain config for GitHub Pages. Do not edit by hand — it's managed by the Pages settings UI. |
| `/.nojekyll` | Stops GitHub Pages running files through Jekyll. |

Everything outside the portfolio (about, endorsements, resume) has been
deliberately removed from the public surface. Bring them back by reverting
the redirect in `index.html` and restoring nav links.

---

## 2. Tech stack

- **No build step.** Vanilla HTML, CSS, JS. Open files directly or serve
  with `python3 -m http.server 8000`.
- **No framework, no bundler, no npm.** If you need to add a dependency,
  prefer a single `<script>` or `<link>` over introducing a build.
- **Hosting**: GitHub Pages, source branch = `main`, folder = `/ (root)`.
- **Custom domain**: `naveenrajmohan.com` with `Enforce HTTPS` on.

---

## 3. Visual system

All design tokens live as CSS custom properties at the top of `styles.css`.
**Change them there once, not in component rules.**

```
--bg          page background
--bg-muted    section--muted background
--text        primary text
--text-soft   secondary text (descriptions, captions, dates)
--accent      orange (#c2410c light / #fb923c dark) — links, arrows, focus
--border      hairline borders
--card        card surface
--radius      14px corner radius for cards / modal
--shadow      shared card / modal shadow
--maxw        960px container width
--space       1.25rem base spacing unit
```

- **Light + dark mode** are driven by `@media (prefers-color-scheme: dark)`.
  Both palettes are defined alongside `:root`. Never hard-code colours in
  component rules; reference the tokens.
- **Typography**: system font stack, 17px base, line-height 1.6.
  Heading scale: `.display` (clamped 2–3rem) > `h2` 1.75rem > `h3` 1.15rem.
- **Layout primitive**: `.container` (`max-width: var(--maxw)` centered).
  Every section's content sits inside `<div class="container">`.
- **Section primitive**: `.section` (vertical padding 4.5rem) +
  `.section--muted` modifier (alt background + top/bottom borders).
  Alternate the modifier between adjacent sections for rhythm.
- **Motion**: hover states use `transform: translateY(-2px)` + shadow
  bump, 150ms ease. Animated marquee respects `prefers-reduced-motion`.

---

## 4. Page anatomy

Every page follows this skeleton. Copy it verbatim when adding a new page:

```html
<header class="site-header">
  <div class="container header-inner">
    <a href="portfolio.html" class="brand">Naveen Raj Mohan</a>
    <nav class="nav" aria-label="Primary">
      <a href="portfolio.html">Portfolio</a>
    </nav>
  </div>
</header>

<main>
  <section class="section">
    <div class="container">
      <a class="back-link" href="portfolio.html">&larr; Back to portfolio</a>
      <h1 class="display">Page Title</h1>
      <p class="lede">One-sentence framing.</p>
    </div>
  </section>

  <!-- alternate .section and .section--muted as you stack content -->
</main>

<footer class="site-footer">
  <div class="container">
    <small>&copy; <span id="year"></span> Naveen Raj Mohan. Built with plain HTML &amp; CSS.</small>
  </div>
</footer>

<script>document.getElementById("year").textContent = new Date().getFullYear();</script>
```

**Header rules:**

- Brand always links to `portfolio.html`.
- Nav has exactly one link: `Portfolio` → `portfolio.html`.
- Do **not** add About, Endorsements, or Resume links until the home page
  is re-enabled. Consistency is the point.

**Back-link rules:**

- Sub-pages always show `← Back to portfolio` pointing at `portfolio.html`.
- Use the existing `.back-link` class — don't restyle.

---

## 5. Component catalogue

### 5.1 Power BI card (`.pbi-card`) — primary portfolio tile

```html
<div
  class="pbi-card"
  role="button"
  tabindex="0"
  data-embed-url="https://app.powerbi.com/view?r=<TOKEN>"
  data-title="Report Title"
  aria-label="Open Report Title in an overlay"
>
  <span class="thumb">
    <img
      class="thumb-image"
      src="assets/report-slug.png"
      alt="Preview of Report Title"
      loading="lazy"
    />
  </span>
  <span class="pbi-body">
    <span class="pbi-title">Report Title <span class="arrow" aria-hidden="true">&rarr;</span></span>
    <span class="pbi-desc">One-sentence description of audience + value.</span>
    <span class="pill pill--accent">Power BI</span>
  </span>
</div>
```

Why a `<div role="button">` and not a `<button>`: nesting `<iframe>` or
`<img>` inside `<button>` is invalid; div + role + tabindex + Enter/Space
handler in JS gives the same UX with valid HTML.

**Thumbnail rules:**

- Static PNG in `assets/`, **16:9 aspect ratio**, ≤ 300 KB.
- Capture the report content only — skip the Power BI chrome bar.
- Filename: kebab-case, no spaces (e.g. `sales-comparison.png`). Spaces
  work via URL-encoding but are a foot-gun.
- The CSS sets `object-fit: cover` on `.thumb-image`, so off-ratio images
  will be cropped at the long edge.

**Card click flow:**

- Click anywhere on the card → opens `#pbi-modal` and sets its `iframe.src`
  from `data-embed-url`, title from `data-title`.
- Modal closes on `×` button, backdrop click, or `Escape`.
- The thumb `<img>` has no `pointer-events: none` because it doesn't catch
  events; only iframes need that.

### 5.2 Modal (`.pbi-modal`)

One per page, lives at the bottom of `<body>`. **Don't duplicate it** —
it's shared by all Power BI cards on the page. Wiring lives in the inline
`<script>` block in `portfolio.html`.

### 5.3 Work card (`.work-card.work-card--link`) — secondary "more work" tile

Use for cross-links to other sub-pages (dbt Estate, Case Studies):

```html
<a class="work-card work-card--link" href="dbt-estate.html">
  <h3>dbt Estate <span class="arrow" aria-hidden="true">&rarr;</span></h3>
  <p>An overview of dbt projects — modelling patterns, lineage, testing, docs.</p>
  <span class="pill">Coming soon</span>
</a>
```

Do **not** use `.work-card` for Power BI reports — those are `.pbi-card`.
Categorisation matters; mixing them dilutes the grid hierarchy.

### 5.4 Project entry (`.project`) — used on `dbt-estate.html`, future case studies

```html
<article class="project">
  <header class="project-head">
    <h2>Project Name</h2>
    <a
      class="btn-ghost"
      href="https://github.com/<owner>/<repo>"
      rel="noopener"
      target="_blank"
    >View on GitHub &rarr;</a>
  </header>

  <!-- Description here. Multiple <p>, lists, code blocks, images all fine. -->
</article>
```

Multiple projects on the same page stack vertically; the CSS adds a divider
between entries automatically. First entry's top divider is suppressed.

### 5.5 Pills

- `.pill` — neutral label ("Coming soon").
- `.pill--accent` — accent-tinted label ("Power BI", "dbt", "Case Study").

Use pills sparingly — one per card maximum. They're status / category
markers, not decoration.

### 5.6 Ghost button (`.btn-ghost`)

Bordered rounded link, hover changes border + text to accent. Use for
secondary actions (e.g. `View on GitHub`). Primary CTAs don't exist on
this site yet; if you add one, add the rule here first.

---

## 6. Content recipes

### Add a Power BI report

1. In Power BI: File → Embed Report → **Publish to web (public)**.
   Copy the iframe `src` URL.
2. Take a 16:9 screenshot of the report content (no chrome). Save to
   `assets/<report-slug>.png`. Keep under 300 KB.
3. In `portfolio.html`, duplicate the last `.pbi-card` block inside
   `.pbi-grid` and update:
   - `data-embed-url` (modal target)
   - `data-title` (modal heading)
   - `aria-label` (`"Open <title> in an overlay"`)
   - `<img src>`, `<img alt>`
   - `.pbi-title`, `.pbi-desc`
4. Test locally (`python3 -m http.server`), confirm the card opens the
   modal and the iframe loads.

### Add a project to `dbt-estate.html`

1. Duplicate an existing `<article class="project">` block.
2. Update `<h2>`, the GitHub link `href`, and the description body.
3. Order: most recent at the top. Dividers come from CSS — no extra HTML.

### Add a sub-page

1. Copy `case-studies.html` as a starting skeleton.
2. Update `<title>`, `<meta name="description">`, the `<h1 class="display">`,
   and the lede.
3. Stick to the section / section--muted alternation.
4. Link to it from `portfolio.html`'s "More work" grid via a
   `.work-card.work-card--link`.
5. Do not add it to the header nav — keep nav at one link.

### Add a new image asset

- Put it in `assets/`.
- Kebab-case filename, no spaces. PNG for screenshots, JPG/WebP for
  photo-like content.
- Include descriptive `alt` text in every `<img>`.

---

## 7. Accessibility conventions

- **Headings**: every page has exactly one `<h1>`. `<h2>` for major
  sections, `<h3>` for cards inside a section.
- **Focus**: never remove the focus ring globally; existing components
  use `:focus-visible` with an accent border. Match this for new
  interactive elements.
- **Keyboard**: any `role="button"` element must respond to both `Enter`
  and `Space` (see the `pbi-card` keydown handler in `portfolio.html`).
- **`aria-label`** on icon-only or thumbnail-only triggers; informative
  text labels otherwise.
- **Modals** use `role="dialog" aria-modal="true" aria-labelledby="<id>"`,
  restore focus to the opener on close, and lock body scroll while open.
- **Iframes** always have a `title` attribute and a sensible `referrerpolicy`.
- **Reduced motion**: any new animation must check
  `@media (prefers-reduced-motion: reduce)` and disable / soften.

---

## 8. Deployment

- **Source of truth**: `main` branch, `/ (root)` folder.
- **Pages settings**: Repo → Settings → Pages → Source = `Deploy from a
  branch`, branch = `main`, folder = `/ (root)`.
- **Custom domain**: managed by GitHub Pages settings. The `CNAME` file
  (containing `naveenrajmohan.com`) is auto-committed by GitHub when the
  custom domain is set. **Do not delete it manually.**
- **DNS** (managed at GoDaddy):
  - `A @` → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`,
    `185.199.111.153` (one A record, four values).
  - `CNAME www` → `datagandhi.github.io.`
- Pages rebuild takes 30–90 s after each push to `main`. Check progress
  in the **Actions** tab under "pages build and deployment".

### Local development

```bash
python3 -m http.server 8000
# open http://localhost:8000/
```

The `index.html` redirect won't fire in `file://` previews because of
browser security; use the HTTP server.

---

## 9. Don't-do list

- **Don't add a build step / npm / framework.** This site's value is that
  it loads instantly and is editable by anyone in any text editor.
- **Don't hard-code colours.** Reference CSS variables.
- **Don't duplicate the modal.** One per page.
- **Don't introduce parallel card components.** Extend `.pbi-card`,
  `.work-card`, or `.project` — or add a fifth pattern to this README
  before using it.
- **Don't add tracking, analytics, or third-party JS** without first
  considering whether a static log line in the GitHub Pages access logs
  would do.
- **Don't put real resume PII back on the public page** without confirming
  the intent — the redirect on `index.html` was added on purpose.

---

## 10. When in doubt

Re-read this README, then look at how an existing component does it.
The site is small enough that grepping for the class name will tell you
everything you need in seconds.