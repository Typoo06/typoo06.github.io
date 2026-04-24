# AGENTS.md

## Project Intent

This repository is an Astro site based on the Fuwari template. Default to incremental improvements, not broad rewrites.

Primary direction for this fork:

- Keep Fuwari aesthetics: soft, clean, lightweight, personal.
- Shift toward a professional public security portfolio/blog.
- Maintain approximately 80% professional tone and 20% subtle cute accents.
- Preserve compatibility with existing Fuwari features unless replacement is explicitly requested.

## Source Of Truth

- Project setup and features: [README.md](README.md)
- Contribution workflow and scope discipline: [CONTRIBUTING.md](CONTRIBUTING.md)
- Localized docs: [docs/README.es.md](docs/README.es.md), [docs/README.id.md](docs/README.id.md), [docs/README.ja.md](docs/README.ja.md), [docs/README.ko.md](docs/README.ko.md), [docs/README.th.md](docs/README.th.md), [docs/README.vi.md](docs/README.vi.md), [docs/README.zh-CN.md](docs/README.zh-CN.md)

Link to existing docs instead of duplicating them.

## High-Value Project Map

- Site config and profile: [src/config.ts](src/config.ts)
- Content schema: [src/content/config.ts](src/content/config.ts)
- Routes and page composition: [src/pages](src/pages) and [src/layouts](src/layouts)
- UI components: [src/components](src/components)
- Search implementation: [src/components/Search.svelte](src/components/Search.svelte), [src/components/Navbar.astro](src/components/Navbar.astro), [pagefind.yml](pagefind.yml)
- Taxonomy utilities: [src/utils/content-utils.ts](src/utils/content-utils.ts), [src/utils/url-utils.ts](src/utils/url-utils.ts)
- Styles and theme variables: [src/styles](src/styles), [src/layouts/Layout.astro](src/layouts/Layout.astro)

## Required Workflow For Agents

When implementing feature changes:

1. Inspect relevant existing files and explain current structure before editing.
2. Propose a short step-by-step plan.
3. Implement in small batches (prefer editing existing Fuwari files over creating parallel systems).
4. After each batch, run relevant checks and summarize changes.
5. Report remaining issues clearly.

## Commands

Use pnpm only.

- Install: `pnpm install`
- Dev: `pnpm dev`
- Check: `pnpm check`
- Type check: `pnpm type-check`
- Format: `pnpm format`
- Lint/fix: `pnpm lint`
- Build (includes Pagefind): `pnpm build`
- Preview build: `pnpm preview`

Run `pnpm build` for changes touching routes, content, search, or rendering behavior.

## Implementation Guardrails

- Keep output static and GitHub Pages friendly.
- Avoid unnecessary client-side JavaScript; prefer Astro/server-rendered output with minimal islands.
- Maintain semantic HTML, accessibility, responsive layout, and reduced-motion friendliness.
- Do not break dark mode, mobile layout, or existing navigation/search/archive behavior.
- Avoid gimmicky widgets and visual clutter.
- Remove dead code introduced by the change.
- Avoid one-off hacks; if unavoidable, document rationale near the change.

## Content And Visual Direction

- Intended audience: pentester and bug bounty learner.
- Tone: professional, calm, minimal, technically credible.
- Cute elements are allowed only as subtle accents (avatar, favicon, 404 illustration, empty states, tiny stickers).
- Do not shift the site into an anime fan style.

## Search And Taxonomy Requirements

- Keep or improve the current search experience before replacing the search engine.
- If search quality is weak, improve UX first (result clarity, filtering cues, navigation) while keeping Pagefind compatibility.
- Prioritize discoverability via title, tags, category, and description.
- If adding a dedicated search page, keep it clean and compatible with static build.
- Maintain clear archive/listing pages.
- Use security-blog-friendly categories consistently: `Labs`, `Writeups`, `Theory`, `Notes`, `Projects`.
- Keep tags supported and cleanly surfaced in existing archive/navigation patterns.

## Compatibility Notes

- Post frontmatter uses `lang` in content schema. Keep metadata aligned with [src/content/config.ts](src/content/config.ts).
- Category/tag archive links rely on query parameters under `/archive/`. Preserve this contract unless intentionally migrated across all related files.
- Search in development uses mock behavior; validate real search via production build and preview.
