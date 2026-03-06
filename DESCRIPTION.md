# Project File Structure

## Vite-Generated Files

### `index.html` — The entry point of everything

Unlike Hugo where templates generate HTML, in Vite **this single HTML file is the starting point**. Notice two key things:
- `<div id="root"></div>` — an empty container where React will mount your entire app
- `<script type="module" src="/src/main.tsx"></script>` — tells the browser to load your React code

Vite serves this file for every request during development. It's like the shell that React fills in.

### `src/main.tsx` — The React bootstrap

This is where React "takes over" the HTML page:
```tsx
createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```
It finds that `<div id="root">` from `index.html` and renders the `<App />` component into it. `StrictMode` is a React wrapper that helps catch common mistakes during development (double-renders, deprecated APIs, etc.).

### `src/App.tsx` — Your actual application

This is the React component you saw in the browser — the logos, the counter button. This is what we'll gut and replace with your site's layout, router, and pages. Right now it's just demo code.

### `src/App.css` + `src/index.css` — Stylesheets

- `index.css` — global styles (loaded in `main.tsx`)
- `App.css` — styles specific to the App component

We'll replace both with Tailwind CSS later.

### `src/assets/react.svg` + `public/vite.svg` — Static assets

Two different places for assets, and this is important to understand:
- **`src/assets/`** — Assets that get **processed by Vite** (hashed filenames, optimized). You import them in code like `import logo from './assets/react.svg'`
- **`public/`** — Assets copied **as-is** to the build output, no processing. Referenced by absolute path like `/vite.svg`. This is where your `CNAME` will go.

### `vite.config.ts` — Vite's configuration

```ts
export default defineConfig({
  plugins: [react()],
})
```
Minimal right now — just the React plugin (handles JSX transformation, fast refresh). We'll add Tailwind and MDX plugins here later.

### `package.json` — Dependencies and scripts

The scripts are:
- **`dev`** -> `vite` — starts the dev server (what you just ran)
- **`build`** -> `tsc -b && vite build` — first type-checks with TypeScript, then bundles for production
- **`preview`** -> `vite preview` — serves the production build locally (useful for testing)
- **`lint`** -> `eslint .` — checks code quality

### `tsconfig.json` — TypeScript config (root)

This is a **project references** setup — it doesn't configure anything itself, it just points to two sub-configs:
- `tsconfig.app.json` — for your app code in `src/`
- `tsconfig.node.json` — for Node-side code (just `vite.config.ts`)

**Why two?** Your app runs in the browser (needs DOM types), but `vite.config.ts` runs in Node (needs Node types). They're different environments with different available APIs.

### `tsconfig.app.json` — TypeScript for your app

Key settings:
- `"jsx": "react-jsx"` — enables JSX/TSX syntax
- `"strict": true` — full type-checking (catches more bugs)
- `"noEmit": true` — TypeScript only checks types, Vite handles the actual compilation
- `"include": ["src"]` — only checks files in `src/`

### `tsconfig.node.json` — TypeScript for Node-side code

Same strict settings but targets Node's environment. Only covers `vite.config.ts`.

### `eslint.config.js` — Linting rules

Checks your code for common mistakes. Includes React-specific rules (hooks rules, refresh safety).

### `.gitignore` — Tells git what to skip

Ignores `node_modules/`, `dist/` (build output), and other generated files.

### `pnpm-lock.yaml` — Exact dependency versions

Ensures everyone installing the project gets the exact same dependency tree. Never edit manually.

## Pre-Migration Files

| File | Status |
|---|---|
| `content/posts/chile-public-access-api.md` | Blog post — will become MDX |
| `content/archives.md` | Hugo archives page — will become a React component |
| `CNAME` | Needs to move into `public/` |
| `.github/workflows/gh-pages.yml` | Old Hugo workflow — will be replaced |
| `CLAUDE.md`, `MIGRATION.md`, `TODO.md` | Planning docs |
| `README.md` | Vite overwrote this with its own — update later |

## How It All Connects

```
Browser requests page
  -> index.html loads
    -> <script> loads src/main.tsx
      -> main.tsx renders <App /> into <div id="root">
        -> App.tsx is your whole application
```

Everything flows from `index.html` -> `main.tsx` -> `App.tsx`. That's the chain you'll build on.
