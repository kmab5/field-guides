# Repository Instructions

This repository is a zero-build static site for publishing YTB yatay geçiş guides.
Use [README.md](README.md) as the primary orientation document and keep instructions here short, actionable, and specific to agent work.

## What matters here

- Treat [guides/informal/yatay-gecis.md](guides/informal/yatay-gecis.md) and [guides/formal/yatay-gecis.md](guides/formal/yatay-gecis.md) as the main content files.
- The informal guide is the source of truth for shared metadata in [guides.json](guides.json): `title`, `description`, and `order` come from the informal version first.
- The site is rendered entirely on the client from Markdown in [assets/js/app.js](assets/js/app.js) and styles in [assets/css/styles.css](assets/css/styles.css).

## Markdown and rendering rules

- Prefer normal Markdown over raw HTML. The renderer already handles headings, lists, tables, links, code, blockquotes, and external-link behavior.
- Use `##` and `###` for page structure. Those headings become the on-page table of contents automatically.
- Do not add manual TOC markup or heading anchors; the app generates both.
- Use callout containers only with the supported syntax: `:::note`, `:::tip`, `:::warning`, and `:::important`.
- Use a leading `tl;dr:` paragraph when you want the page to render a summary callout.
- Use a leading `PS.` or `ps:` paragraph when you want the quiet postscript styling.
- Avoid leaving an empty trailing `##` at the end of a guide; the renderer ignores it and it usually means unfinished structure.
- Keep tables compact and readable; the UI wraps them in a horizontal scroll container automatically.
- External links should be real URLs. The app opens them in a new tab automatically.

## Content style for this project

- Match the existing voice and register of the guide being edited. Keep informal content conversational; keep formal content polished and direct.
- If you are editing guide prose, preserve the author’s intent and formatting patterns unless the task explicitly asks for a rewrite.
- Put front matter at the top of each Markdown file when needed: `title`, `description`, `order`, `updated`, and `wip` are the supported fields.
- If you add a new guide pair, keep the filename identical in both `guides/informal/` and `guides/formal/`.

## Validation

- After changing Markdown or manifest logic, validate by regenerating the manifest with `node scripts/build-manifest.mjs` when needed.
- For rendering or navigation changes, serve the repo over HTTP rather than using `file://`.

## Good targets for future customization

- `/create-instruction` for markdown authoring conventions specific to guide writing.
- `/create-skill` for a guide-review workflow that checks front matter, callouts, TOC headings, and readability.
- `/create-agent` for a focused markdown formatting reviewer.
