# Velti

Minimal pnpm workspace scaffold with:

- `apps/web`: Svelte 5 + TypeScript + Tailwind CSS v4 app
- `tools/desktop-export`: placeholder package for desktop export tooling
- `docs/architecture/editor-core.md`: core architecture for the hybrid visual/code editor

## Tailwind plugin selection

The web app currently enables:

- `@tailwindcss/typography`
- `@tailwindcss/forms`

## Scripts

- `pnpm dev` — run the web app in development mode.
- `pnpm build` — type-check and build the web app.
- `pnpm check` — run Svelte type diagnostics.
- `pnpm lint` — run ESLint.
- `pnpm format` — run Prettier.
