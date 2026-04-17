# personal-site

A simple static personal site — bio, LinkedIn endorsements, and a portfolio
section for dashboards and dbt work (coming soon). Plain HTML/CSS, no build
step, hosted on GitHub Pages.

## Run locally

From the repo root:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000/ in a browser. You can also just double-click
`index.html`.

## Edit

- Content lives in `index.html`. Replace every `Your Name`, `Endorser One`,
  and placeholder paragraph with your own copy.
- Styling is in `styles.css`. Colors, spacing, and max width are CSS variables
  at the top — tweak once, applied everywhere.
- Drop images (avatar, screenshots) into `assets/` and reference them from
  `index.html`.

## Publish on GitHub Pages

1. Push to `main`.
2. On GitHub: **Settings → Pages**.
3. Source: **Deploy from a branch**. Branch: `main`. Folder: `/ (root)`. Save.
4. Site goes live at `https://datagandhi.github.io/personal-site/` within a
   minute or two.

The empty `.nojekyll` file ensures GitHub Pages serves files as-is without
running them through Jekyll.
