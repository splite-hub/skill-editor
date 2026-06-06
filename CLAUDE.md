# CLAUDE.md — skill-editor

## Project Overview

This is a **Fresh v2 (Deno) web application** scaffolded as a server-rendered app with interactive client-side islands. The application code lives entirely under `service/`.

**Stack:** Deno · Fresh 2 · Preact · Preact Signals · Tailwind CSS 4 · Vite

---

## Repository Layout

```
skill-editor/
└── service/           # Main application (all dev work happens here)
    ├── routes/        # File-based routing (server-rendered pages + API)
    │   ├── _app.tsx   # Root HTML shell wrapping every page
    │   ├── index.tsx  # Home page  (GET /)
    │   └── api/
    │       └── [name].tsx  # Dynamic API route  (GET /api/:name)
    ├── islands/       # Interactive Preact components (hydrated on the client)
    │   └── Counter.tsx
    ├── components/    # Shared, non-interactive UI components
    │   └── Button.tsx
    ├── assets/        # CSS and other assets processed by Vite
    │   └── styles.css
    ├── static/        # Public static files served as-is
    │   ├── favicon.ico
    │   └── logo.svg
    ├── main.ts        # App entry point: middleware, programmatic routes, fsRoutes()
    ├── client.ts      # Client entry: imports global CSS for HMR
    ├── utils.ts       # Shared State interface + `define` helper
    ├── vite.config.ts # Vite plugins (Fresh + Tailwind)
    └── deno.json      # Tasks, import map, compiler options
```

---

## Development Commands

All commands are run from inside `service/` with `deno task <name>`:

| Command | What it does |
|---|---|
| `deno task dev` | Start Vite dev server with HMR |
| `deno task build` | Production build → `_fresh/` |
| `deno task start` | Serve the production build (`_fresh/server.js`) |
| `deno task check` | Format check + lint + type-check (run before committing) |
| `deno task update` | Upgrade Fresh and its plugins |

**`deno task check` must pass before every commit.** It runs:
1. `deno fmt --check .` — formatting enforced by Deno's built-in formatter
2. `deno lint .` — linting with `fresh` + `recommended` rule tags
3. `deno check` — full TypeScript type checking

---

## Architecture Patterns

### Routes vs Islands vs Components

| Location | Rendered where | Has interactivity |
|---|---|---|
| `routes/` | Server only | No (use `<IslandComponent />` for interactivity) |
| `islands/` | Server + client (hydrated) | Yes |
| `components/` | Server only | No |

- Keep business logic in routes and utility modules.
- Islands are for UI interactivity only; they receive `Signal` props from routes.
- Never import an island from a component — islands must be leaf nodes.

### Shared State via Middleware

`utils.ts` defines the `State` interface. Add any cross-route shared values there:

```ts
// utils.ts
export interface State {
  shared: string;  // add new fields here
}
export const define = createDefine<State>();
```

Populate state in a middleware inside `main.ts`. Access it in routes as `ctx.state`.

### Route Definition

Prefer **file-based routes** in `routes/`. Use programmatic routes in `main.ts` only for lightweight one-liners or middleware-registered endpoints. The file `routes/api/[name].tsx` shows the pattern for dynamic segments.

### Signals for State

Use `@preact/signals` for reactive state. Signals can be created in a route and passed to islands as props:

```ts
// In a route (server-rendered)
const count = useSignal(3);
return <Counter count={count} />;

// In an island (client-hydrated)
props.count.value += 1;  // triggers re-render
```

---

## Styling

- **Tailwind CSS 4** via `@tailwindcss/vite` plugin — utility classes are available everywhere.
- Global styles and theme customization go in `assets/styles.css`.
- The gradient `.fresh-gradient` is defined there and used on the home page.
- No separate Tailwind config file; configuration is handled inline in `styles.css` or via the Vite plugin.

---

## Import Map

Managed in `deno.json` under `"imports"`. Short aliases:

| Alias | Package |
|---|---|
| `fresh` | `jsr:@fresh/core@^2.3.3` |
| `preact` | `npm:preact@^10.29.1` |
| `@preact/signals` | `npm:@preact/signals@^2.9.0` |
| `@/` | `./` (repo root alias) |
| `vite` | `npm:vite@^7.1.3` |

`nodeModulesDir: "manual"` — Deno manages `node_modules` explicitly; do not run `npm install`.

---

## TypeScript & JSX

- JSX mode: `precompile` (Preact-specific optimization — faster than transform mode).
- `jsxImportSource`: `preact`.
- Standard HTML elements listed in `jsxPrecompileSkipElements` are not precompiled (pass through as-is).
- Lib targets: `dom`, `dom.asynciterable`, `dom.iterable`, `deno.ns`.
- Vite client types are included via `"types": ["vite/client"]`.

---

## No Tests

There is currently no test infrastructure. When adding tests:
- Use Deno's built-in test runner (`deno test`).
- Add a `"test"` task to `deno.json`.
- Place test files alongside source files as `*.test.ts` or in a top-level `tests/` directory.

---

## No CI/CD

There are no GitHub Actions workflows. When adding CI, the minimum pipeline should run:

```yaml
- run: deno task check   # format, lint, type-check
- run: deno task build   # ensure production build succeeds
```

---

## Key Conventions

- **Formatting:** Always run `deno fmt` — the formatter is the source of truth, do not manually adjust formatting.
- **File naming:** Routes and islands use `PascalCase.tsx`; utilities use `camelCase.ts`.
- **No barrel files:** Import directly from the source file.
- **`_fresh/` is generated:** Never edit files inside `_fresh/` — they are overwritten on every build and are git-ignored.
- **`.env*` files are git-ignored:** Use Deno's `--env` flag or system env vars for secrets.
- **Avoid `vendor/`:** Dependencies are resolved from the Deno cache or `node_modules` (manual mode); do not commit a vendor directory.

---

## Git

- Development branch for this repo: `claude/claude-md-docs-6QXvy`
- The `main` branch is the stable base; feature work goes on named branches.
- Commit messages should be in English, imperative mood, present tense (e.g., `add counter island`, `fix api route type`).
