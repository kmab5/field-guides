# kmab5 field guides

A small, no-framework static site for publishing student guides. It's built to run on **GitHub Pages** with zero build step.

Every guide can exist in two registers:

- **Informal** — the student's own draft voice (always present).
- **Formal** — a revised, cleaned-up version (added when you're ready).

A switch in the top bar (next to the dark-mode button) flips between them; the reader's accent colour shifts to match — red for informal, teal for formal.

The top bar also has a button to hide/show the sidebar (it collapses the sidebar on desktop and opens the guide drawer on mobile), and the **formal** version of any guide shows a Google Translate bar across the top so readers can translate the page into their own language.

---

## Repository layout

```
.
├── index.html                     # the app shell
├── guides.json                    # generated manifest (list of guides + versions)
├── assets/
│   ├── css/styles.css
│   └── js/
│       ├── app.js                 # routing + markdown rendering
│       └── marked.js              # vendored Markdown parser (marked v12)
├── guides/
│   ├── informal/                  # informal .md files live here
│   │   └── yatay-gecis.md
│   └── formal/                    # formal .md files live here
│       └── yatay-gecis.md
├── scripts/
│   └── build-manifest.mjs         # regenerates guides.json
└── .github/workflows/
    └── build-manifest.yml         # rebuilds guides.json + deploys on push
```

---

## Adding a new guide

1. Write the informal version as a Markdown file in `guides/informal/`, e.g. `guides/informal/burslu-staj.md`.
2. (Optional) Write the formal version at the **same filename** under `guides/formal/`, e.g. `guides/formal/burslu-staj.md`.
3. Commit and push. The GitHub Action rebuilds `guides.json` and redeploys.

**The filename is the pairing key.** `informal/burslu-staj.md` and `formal/burslu-staj.md` are treated as two versions of the *same* guide. A guide can have just an informal version — the formal toggle then shows a friendly "coming soon" until you add the matching file.

### Manually editing manifest

You don't have to use the Action. `guides.json` is a plain file you can edit directly, or you can regenerate it locally:

```bash
node scripts/build-manifest.mjs
```

Both approaches are supported. If you set a `defaultSlug` in `guides.json` by hand, the generator preserves it.

---

## Front matter

Put an optional YAML block at the very top of each `.md` file, between `---` fences:

```markdown
---
title: A Guide to Yatay Geçiş for YTB Students
description: One-line summary shown in the sidebar.
order: 1
updated: 2026-07-21
wip: true
---
```

| Field | Purpose |
|---|---|
| `title` | Sidebar + browser-tab title. Falls back to the first `# heading`, then the filename. The informal file's title is used for the pair. |
| `description` | Short blurb under the title in the sidebar. |
| `order` | Sort position in the sidebar (lower first). Lowest-ordered guide is the default landing page. |
| `updated` | Date shown under the guide title. Falls back to the file's modified date. |
| `wip` | `true` shows the amber "still being written" banner and a dot in the sidebar. |

The parser understands strings, numbers, and `true`/`false` — that's all you need here.

---

## Markdown features

Standard Markdown works (headings, lists, tables, links, code, blockquotes). On top of that:

### Callout containers

Wrap content in `:::type` … `:::` to make a callout. An optional title follows the type on the first line.

```markdown
:::warning Set your expectations early
Transfers often don't go through. Keep your hopes measured.
:::

:::note In short
The transfer you want is **kurumlar arası yatay geçiş**.
:::

:::tip Always go to the source
Read the university's official directive (*yönerge*).
:::

:::important Read this twice
Apply as a 2nd/3rd-year student — never as a credit-transferred first year.
:::
```

Types: `note` (blue), `tip` (teal), `warning` (amber), `important` (red) — anything unrecognized renders as `note`. Each type has its own fixed colour that stays the same in both the informal and formal versions, so they never blur together. The inside is full Markdown.

### tl;dr summaries

Any paragraph that starts with `tl;dr:` automatically becomes a violet summary callout:

```markdown
tl;dr: start worrying around mid-to-end of June, but your GPA matters all year.
```

### Postscripts

A paragraph starting with `PS.` (or `ps:`) is styled as a quiet postscript note.

### Automatic table of contents

Every `##` and `###` heading is collected into the "On this page" rail (visible on wide screens) with scroll tracking. Headings get hover anchors for in-page jumps. An empty trailing `##` is ignored.

### Tables

Standard Markdown tables are wrapped so they scroll horizontally on small screens instead of breaking the layout.

---

## Brand assets

The favicon and logo live in [`public/brand/`](public/brand/): a minimal teal flame (a nod to the "a candle in the dark" tagline). `favicon.svg` is the modern icon, `favicon.ico` / `favicon-32.png` / `apple-touch-icon.png` are fallbacks wired up in `index.html`, and `logo.svg` / `logo.png` are the standalone mark. To restyle, edit the SVGs; the flame in the top bar is inlined in `index.html` and takes its colour from CSS.

## Google Translate on the formal version

The formal version of each guide shows a Google Translate bar at the top of the page. It's the free client-side **Website Translator** widget — there's nothing to sign up for and no API key. The widget script is already in `index.html`; it only needs the site to be served over HTTP (it won't run from `file://`), which GitHub Pages does. A couple of things worth knowing: Google translates the whole visible page, and Google no longer offers new Website Translator registrations, though the existing embed script still works. If it's ever discontinued, the site keeps working — the bar simply won't appear.

## Running locally

Because the site fetches `guides.json` and the `.md` files, open it through a local server rather than `file://`:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

---

### made with love 🫶 [kmab](https://kmab5.github.io)
