# GalaxyBrain — Ryan's Digital Garden

## What This Is

Ryan's personal digital garden published as an interactive 3D knowledge graph. Built on the **GalaxyBrain** template: Astro 5 static site + `react-force-graph-3d` (Three.js). Notes live in `vault/` as Obsidian-flavored Markdown; the site is deployed to Netlify from the `main` branch.

**Live site:** ryankelly.garden (Ryan's garden) — not the demo at galaxybrain.netlify.app.

---

## Key Mental Model

- **Content lives in `vault/*.md`** — edited in Obsidian, committed via Obsidian Git plugin.
- **`publish: true` in frontmatter is the gate** — notes without it are completely invisible: no graph node, no rendered page, no links.
- **Filename = canonical title** — `vault-parser.ts` always overrides the frontmatter `title` with the filename (without `.md`). Rename in Obsidian, and the title updates everywhere.
- **Slugify rule**: filename → lowercase, non-alphanumeric runs → hyphens. `"Books!"` → `books`, `"Digital Garden"` → `digital-garden`.
- **The landing page is the 3D graph** (`/`). Notes are at `/notes/<slug>`.

---

## Garden Architecture (Ryan's Maturity Levels)

From `What is this Garden Thing.md` — the content taxonomy:

- **Seeds** — earliest nugget, little/no context
- **Budding** — active development, not yet standalone
- **Evergreen** — fully formed, readable alone; often becomes a blog post
- **Signpost** — entry points, navigation aids, "desire paths"
- **Rocks** — other people's content; highlights + transclusion of sources
- **Gems** — high-density collections (Rocks + Evergreens + resources); basis for paid courses

Epistemic disclosure is a goal: notes should indicate not just maturity level but confidence level.

---

## Frontmatter Schema

```yaml
---
publish: true                   # REQUIRED to appear anywhere
title: "Note Title"             # Auto-overridden by filename — optional but documents intent
tags: [category/subcategory]    # Hierarchical tags supported (parent/child)
aliases: [alternate-name]       # Alternative names for wikilink resolution
graph:
  shape: sphere                 # sphere|box|cone|cylinder|dodecahedron|torus|torusknot|octahedron
  color: "#3498db"              # Hex color for this node
  collapsible: true             # Starts collapsed; click to reveal downstream nodes
  pinned: true                  # Always visible even when parent is collapsed
  callout: true                 # Show animated "start here" arrow on landing page
  calloutText: "Click to start" # Text in the callout bubble
cover: attachments/image.png    # Optional hero image
---
```

Tags appear as octahedron nodes in the graph; hierarchical tags (`a/b`) create parent→child edges.

---

## Project Structure

```
vault/                  ← Obsidian vault — all content here
  *.md                  ← Top-level notes
  Books Folder/         ← Book-specific notes
  attachments/          ← Images (webp, png, svg, gif)
  template/Template.md  ← Obsidian note template
  .obsidian/            ← Obsidian config (gitignored workspace files)

src/
  components/
    FullGraph.tsx        ← Landing page 3D graph (React island, client:only)
    LocalGraph.tsx       ← Note sidebar graph — same full graph.json as the landing page, camera focused on the current note (React island, client:only)
    GraphNodeFactory.ts  ← shape string → Three.js geometry
  integrations/
    graph-builder.ts     ← Builds graph.json at build time
    asset-collector.ts   ← Copies vault images → public/vault-assets/
    block-indexer.ts     ← Indexes ^block-id markers for transclusion
  layouts/
    BaseLayout.astro     ← HTML shell, anti-FOUC, global theme toggle
    NoteLayout.astro     ← Two-column: prose + LocalGraph sidebar
  lib/
    vault-parser.ts      ← Parses .md files (gray-matter + extraction)
    link-resolver.ts     ← Obsidian wikilink resolution
    remark-wikilinks.ts  ← [[wikilinks]] → <a data-wikilink>
    types.ts             ← All shared TypeScript types
    graph-types.ts       ← NoteGraphData / NoteRef types
    hosting.ts           ← GitHub Pages base path detection
    public-path.ts       ← BASE_URL helper for client-side fetches
  pages/
    index.astro          ← Landing page (mounts FullGraph)
    notes/[...slug].astro ← Dynamic note pages
  plugins/
    remark-vault-images.ts   ← ![[img.ext]] → <img> with vault-assets path
    remark-transclusion.ts   ← ![[note#^id]] → blockquote embeds
  styles/
    global.css           ← CSS variables, typography, theme system
    note.css             ← Prose styles, transclusion styles
    callouts.css         ← Obsidian callout block styles
    graph.css            ← Graph page wrapper styles
  content.config.ts      ← Astro content collection (vault/*.md)

scripts/
  sync-titles.mjs        ← Repairs frontmatter (strips blank lines, syncs title to filename)

public/
  graph.json             ← Full graph data — consumed by both the landing page and the note sidebar (generated, gitignored)
  vault-assets/          ← Copied vault images (generated)
  favicon.svg / .ico     ← Site icons
  GalaxyGIF.dark.gif     ← Demo GIF for README
```

---

## Commands

```bash
pnpm dev          # Dev server at localhost:4321 — rebuilds graph on vault changes
pnpm build        # Production build (runs sync-titles first, then astro build)
pnpm preview      # Serve dist/ locally
pnpm sync-titles  # Repair frontmatter in vault without full build
```

**Netlify build:** `pnpm astro clean && node scripts/sync-titles.mjs && pnpm build`, publish dir `dist/`, Node 22.

---

## Build Pipeline (order matters)

```
astro:config:done
  └── graph-builder → public/graph.json

astro:build:start
  ├── asset-collector → public/vault-assets/, .astro/vault-images.json
  └── block-indexer   → .astro/block-index.json

remark pipeline (per .md render)
  1. remark-vault-images   — ![[img.ext]] → <img src="/vault-assets/...">
  2. remark-transclusion   — ![[note#^id]] / ![[note]] → HTML embeds
  3. remark-wikilinks      — [[note]] → <a href="/notes/slug" data-wikilink>
  4. remark-obsidian       — callouts, ==highlights==, %%comments%%, tasks
  ↓
  rehype-raw               — lets inline HTML from remark through
```

**The remark plugin order is fixed** — changing it in `astro.config.mjs` will break rendering. `remark-vault-images` must consume `![[img]]` before the others see it; `remark-transclusion` must run before `remark-wikilinks` consumes `[[...]]`.

---

## Theme System

- Dark/light via `html.dark` / `html.light` CSS classes.
- Default: dark. Toggle button fixed top-right on every page.
- Anti-FOUC: inline `<script>` in `<head>` sets class before first paint.
- All colors are CSS custom properties — never hardcoded. See `global.css`.
- React graph components listen for `CustomEvent('theme-change')` and `'storage'` events to sync without page reload.
- Shiki dual-themes: `github-dark` / `github-light`, bound via CSS vars on `.astro-code`.
- **`nodeThreeObject` must stay referentially stable** — both graph components use `useRef(isDark)` + `fgRef.current?.refresh?.()` to update colours on theme change without restarting the force simulation. Do not replace this with direct `isDark` state reads inside the callback.

---

## Wikilink Resolution

Obsidian shortest-path resolution:
1. Exact slug match (filename slugified)
2. Case-insensitive filename match
3. Alias match (frontmatter `aliases`)
4. No match → ghost node (visible in graph, grayed out, "Not yet created" on click)

Ghost links in note prose render with dashed underline and `--link-ghost` color.

---

## Graph Interaction Model

**Full graph (landing page):**
- Left-click file node → navigate to note
- Left-click collapsible node (collapsed) → expand, reveal downstream
- Shift+click collapsible node (expanded) → re-collapse
- Left-click tag node → toggle tag filter (highlights connected nodes, fades others)
- Left-click ghost node → "Note not yet created" toast
- Right-click any node → camera fly-to
- `?focus=noteId` URL param → auto-focus camera on load
- `?highlight=tag:name` URL param → activate tag filter on load

**Local graph (note sidebar):**
- Loads the entire vault graph (same `graph.json` as the landing page) — not just the current note's neighbourhood
- Camera auto-flies to and zooms in on the current note once the force simulation settles
- Left-click file node → navigate directly (every node is already visible, so there's no expand step)
- Left-click ghost node → "Note not yet created" toast; tag nodes are a no-op in the 3D view
- Right-click any node → camera fly-to
- Tag pills (in the sidebar list) → `/?highlight=<tagId>` on full graph

**`SHOW_BUILD_CTA` constant** in `FullGraph.tsx` — set to `false` once Ryan's own content replaces template notes.

---

## Content Authoring Notes

- Images go in `vault/attachments/`. Reference as `![[filename.ext]]` in notes.
- Block IDs: append `^my-id` at end of a line. Transclude with `![[Note Name#^my-id]]`.
- Full-note transclusion: `![[Note Name]]` — renders as a collapsible `<details>` block.
- Tags can be in frontmatter `tags:` array or inline `#tag` in body.
- Aliases enable wikilinks by alternate name: `aliases: [Books, AudioBook]` lets `[[Books]]` resolve.

---

## Adding Things

| Task | Files to touch |
|---|---|
| New node shape | `src/lib/types.ts` (union), `src/components/GraphNodeFactory.ts` (geometry case) |
| New frontmatter field | `src/lib/types.ts` (interface), `src/integrations/graph-builder.ts` (mapping + defaults on tag/ghost nodes) |
| New CSS token | `src/styles/global.css` (`html.dark` and `html.light` blocks) |
| New remark plugin | `astro.config.mjs` (add to array, mind the order above) |

---

## Conventions

- TypeScript strict mode — no `any` without a comment explaining why
- Remark plugins must never throw — wrap risky work in try/catch and log a warning
- Graph builder must produce valid JSON even when individual notes are malformed
- No external CSS frameworks — plain CSS custom properties only
- React graph components are `client:only="react"` islands; `react-force-graph-3d` requires browser globals (WebGL, `window`) that don't exist during SSR

---

## Known Issues / Technical Debt

1. **`SHOW_BUILD_CTA = true`** in `FullGraph.tsx` — still showing the "Build your own" prompt. Flip to `false` once content is Ryan's.
2. **No search** — Pagefind integration would add full-text search at zero JS overhead.
3. **Vault raster images** bypass Astro's WebP/AVIF pipeline. Could add `sharp` post-processing in `asset-collector` for PNG/JPG if performance matters.

---

## Design Goals (Ryan's stated intent)

- Non-linear, non-chronological notes — wandering is encouraged
- "Garage door open" learning: transparent, in-progress thinking
- Rubber ducking: writing to an imagined audience aids insight
- Mix of paid-course-quality "Gems" and raw "Seeds" — honest about maturity
- Old weird internet vibes: you have to figure out the interface
- Simple navigation: the graph IS the navigation
