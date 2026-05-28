# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

| Command | Description |
|:--------|:------------|
| `pnpm dev` | Start dev server at `localhost:4321` |
| `pnpm build` | Production build to `./dist/`, then run Pagefind indexing |
| `pnpm preview` | Preview the production build locally |
| `pnpm check` | Run `astro check` for template/type errors |
| `pnpm type-check` | Run `tsc --noEmit --isolatedDeclarations` |
| `pnpm format` | Format source with Biome |
| `pnpm lint` | Lint and auto-fix source with Biome |
| `pnpm new-post <filename>` | Scaffold a new post in `src/content/posts/` |

**Package manager**: pnpm only (enforced via `preinstall` script). Use `pnpm add` / `pnpm add -D` to add dependencies.

## Architecture

This is a static blog built on the **Fuwari** template, using **Astro 5** (SSG mode) with **Svelte 5** interactive islands, **Tailwind CSS 3**, and **Swup** for SPA-like page transitions.

### Content

- **`src/content/posts/`** — Blog posts as Markdown/MDX files with frontmatter (`title`, `published`, `tags`, `category`, `draft`, `image`, etc.). Schema defined in `src/content/config.ts`.
- **`src/content/spec/`** — Static content pages (e.g., `about.md`).
- In dev, drafts are visible. In production builds (`import.meta.env.PROD`), posts with `draft: true` are excluded.

### Routing

Astro file-based routing under `src/pages/`:
- **`[...page].astro`** — Paginated homepage (page size: 8, constant in `src/constants/constants.ts`).
- **`posts/[...slug].astro`** — Individual blog post. Renders markdown via `Content` component, applies next/prev post navigation.
- **`about.astro`** — About page from spec collection.
- **`archive.astro`** — Archive with tag/category filtering.
- **`rss.xml.ts`**, **`robots.txt.ts`** — Generated endpoints.

### Layout system

- **`Layout.astro`** — Root HTML shell: `<head>` with meta tags, favicon, theme/hue initialization (inline script before paint), OverlayScrollbars, PhotoSwipe lightbox setup, and Swup hooks. All page-level JS lives here.
- **`MainGridLayout.astro`** — Extends Layout with navbar, banner (parallax), sidebar (Profile + Categories + Tags), TOC, and footer. Provides the main grid: `[sidebar] [main content]`.

### Components (hybrid Astro + Svelte)

**Astro components** (server-rendered, in `src/components/`):
- `PostCard.astro`, `PostMeta.astro`, `PostPage.astro` — Post listing and metadata display.
- `Navbar.astro`, `Footer.astro` — Site chrome.
- `widget/` — Sidebar widgets: `Profile.astro`, `Categories.astro`, `Tags.astro`, `TOC.astro`, `NavMenuPanel.astro`.
- `control/` — `Pagination.astro`, `BackToTop.astro`, `ButtonLink.astro`, `ButtonTag.astro`.
- `misc/` — `Markdown.astro` (styling wrapper), `ImageWrapper.astro`, `License.astro`.
- `ConfigCarrier.astro` — Passes server-side config to client-side Svelte components via data attributes.
- `GlobalStyles.astro` — Global CSS includes.
- `Search.svelte`, `LightDarkSwitch.svelte`, `DisplaySettings.svelte`, `ArchivePanel.svelte` — Interactive islands.

### Markdown processing pipeline

Remark plugins (run in order, configured in `astro.config.mjs`):
1. `remark-math` — Parse math syntax
2. `remark-reading-time` — Inject word count and reading time into frontmatter
3. `remark-excerpt` — Extract excerpt
4. `remark-github-admonitions-to-directives` — Convert GitHub-style admonitions to directives
5. `remark-directive` — Parse `:::` directive syntax
6. `remark-sectionize` — Wrap sections
7. `remark-directive-rehype` — Convert directive nodes for rehype

Rehype plugins:
1. `rehype-katex` — Render math to HTML
2. `rehype-slug` — Add IDs to headings
3. `rehype-components` — Render custom components (GitHub cards, admonitions: note/tip/important/caution/warning)
4. `rehype-autolink-headings` — Append anchor links to headings

Custom plugins live in `src/plugins/`.

### Styling

- **Tailwind** with `@tailwindcss/typography` plugin, dark mode via `class` strategy.
- **Stylus** (`.styl`) files for markdown extensions and CSS variables.
- Global CSS in `src/styles/`: `main.css`, `markdown.css`, `photoswipe.css`, `transition.css`, `scrollbar.css`, `expressive-code.css`.
- Theme hue is a CSS variable (`--hue`) set from `siteConfig.themeColor.hue`, persisted in localStorage.
- PostCSS config uses `postcss-import` and `postcss-nesting`.

### Search

**Pagefind** indexes the built `dist/` folder after each `pnpm build`. Configuration in `pagefind.yml` (excludes KaTeX spans, search panel, pagefind-ignore elements). The `Search.svelte` component provides the UI.

### Configuration

All site-level config is in **`src/config.ts`**: site metadata, theme hue, banner, TOC depth, navbar links, profile, license, and Expressive Code theme. Types are in `src/types/config.ts`.

### i18n

Translation dictionary in `src/i18n/languages/`. Site language set via `siteConfig.lang` in `src/config.ts`. Individual posts can override with `lang` frontmatter. Keys are the `I18nKey` enum in `src/i18n/i18nKey.ts`.

### Path aliases (tsconfig.json)

| Alias | Path |
|:------|:-----|
| `@/*` | `src/*` |
| `@components/*` | `src/components/*` |
| `@assets/*` | `src/assets/*` |
| `@constants/*` | `src/constants/*` |
| `@utils/*` | `src/utils/*` |
| `@i18n/*` | `src/i18n/*` |
| `@layouts/*` | `src/layouts/*` |
