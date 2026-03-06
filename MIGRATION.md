# Migration Plan: Hugo to Vite + React + shadcn/ui

## Overview

Migrate `self.rosebush.garden` from a Hugo static site (PaperMod theme) to a custom-built static site using Vite, React, shadcn/ui, and MDX for content authoring. The live site remains untouched on `main` until the new site is fully ready.
ALSO this will be a teaching opportunity for learning web, React, Shadcn, etc.

---

## Current State

| Aspect         | Detail                                        |
|----------------|-----------------------------------------------|
| Generator      | Hugo with PaperMod theme (git submodule)      |
| Content        | 1 post (`chile-public-access-api.md`), archives page |
| Deployment     | GitHub Actions -> `gh-pages` branch           |
| Domain         | `self.rosebush.garden` (CNAME)                |
| Theme          | Dark mode, profile mode, social links (LinkedIn, GitHub) |

## Target State

| Aspect         | Detail                                        |
|----------------|-----------------------------------------------|
| Build tool     | Vite                                          |
| UI framework   | React 18+ with TypeScript                     |
| Components     | shadcn/ui (Tailwind CSS) with Base-UI instead of Radix                      |
| Content        | MDX files (superset of Markdown, supports JSX components) |
| Routing        | React Router (static pre-rendering)           |
| SSG            | Custom pre-render build step or vite-ssg plugin |
| Deployment     | GitHub Actions -> GitHub Pages (same domain)  |

---

## Git Strategy

### Branch Plan

```
main                  (LIVE - Hugo site, untouched until final cutover)
  |
  +-- migrate/vite-react   (all new development happens here)
```

### Rules

1. **Never push breaking changes to `main`** until the new site is verified.
2. All migration work happens on `migrate/vite-react`.
3. The existing GitHub Actions workflow on `main` continues deploying the Hugo site.
4. The migration branch has its own workflow file (disabled on `main` or gated behind branch conditions).
5. Test the new site using Vite's dev server locally and preview deploys before merging.

### Cutover Process

1. Verify the new site builds correctly (`pnpm build` produces `dist/`).
2. Verify all existing content is present and renders correctly.
3. Verify dark mode, profile section, social links, and archives work.
4. Merge `migrate/vite-react` into `main` via PR.
5. The updated GitHub Actions workflow deploys the Vite build instead of Hugo.
6. Verify `self.rosebush.garden` serves the new site.
7. After confirming, delete the migration branch.

---

## Migration Phases

### Phase 1: Scaffold the New Project

**Branch:** `migrate/vite-react`

1. Create the branch:
   ```bash
   git checkout -b migrate/vite-react
   ```

2. Initialize the Vite + React + TypeScript project at the repo root:
   ```bash
   pnpm create vite@latest . --template react-ts
   ```
   This will create the standard Vite scaffold. Accept overwriting since we'll reorganize.

3. Install core dependencies:
   ```bash
   pnpm add react-router-dom
   pnpm add -D tailwindcss @tailwindcss/vite @tailwindcss/typography
   ```

4. Initialize and install shadcn/ui (uses Base UI primitives, not Radix):
   ```bash
   pnpx shadcn@latest init
   ```
   Choose: TypeScript, dark default theme, CSS variables, Tailwind.

5. Install MDX support:
   ```bash
   pnpm add -D @mdx-js/rollup @mdx-js/react remark-frontmatter remark-mdx-frontmatter rehype-highlight
   ```

6. **Resulting project structure:**
   ```
   /
   ├── content/
   │   └── posts/
   │       └── chile-public-access-api.mdx    # migrated from .md
   ├── src/
   │   ├── components/
   │   │   └── ui/              # shadcn/ui components
   │   ├── layouts/
   │   │   ├── RootLayout.tsx
   │   │   ├── PostLayout.tsx
   │   │   └── ArchiveLayout.tsx
   │   ├── pages/
   │   │   ├── Home.tsx
   │   │   ├── Posts.tsx
   │   │   ├── Post.tsx
   │   │   └── Archives.tsx
   │   ├── lib/
   │   │   ├── content.ts       # content loading utilities (frontmatter parsing, etc.)
   │   │   └── utils.ts         # shadcn/ui utility (cn helper)
   │   ├── App.tsx
   │   ├── main.tsx
   │   └── index.css            # Tailwind + custom styles
   ├── public/
   │   └── CNAME                # preserve custom domain
   ├── index.html
   ├── vite.config.ts
   ├── tailwind.config.ts
   ├── tsconfig.json
   ├── components.json          # shadcn/ui config
   ├── package.json
   └── pnpm-lock.yaml
   ```

7. Remove Hugo artifacts:
   - Delete `config.toml`
   - Delete `archetypes/`
   - Delete `themes/` submodule reference and `.gitmodules`
   - Delete `.hugo_build.lock`
   - Delete old `public/` directory (Vite outputs to `dist/`)

### Phase 2: Content Pipeline

1. **Rename `.md` to `.mdx`** for existing content files (or keep `.md` and configure the MDX plugin to handle both).

2. **Frontmatter handling:**
   - Existing Hugo frontmatter (`title`, `date`, `draft`) carries over directly.
   - Use `remark-frontmatter` + `remark-mdx-frontmatter` to parse YAML frontmatter and expose it as an export.

3. **Content loading strategy using Vite's `import.meta.glob`:**
   ```ts
   // src/lib/content.ts
   const postModules = import.meta.glob('/content/posts/*.mdx', { eager: true });
   ```
   This gives build-time access to all posts with their frontmatter and rendered content.

4. **MDX component mapping:**
   - Provide custom components for standard Markdown elements (headings, links, code blocks).
   - Use shadcn/ui components (Card, Badge, etc.) to style posts.
   - Use `rehype-highlight` or shadcn-compatible code block styling for syntax highlighting.

### Phase 3: Build the Pages

1. **Home page (`/`):**
   - Profile section: name, subtitle ("If we can call it that"), social icons.
   - Replicate the PaperMod profile mode using shadcn/ui components.
   - Social links: LinkedIn, GitHub (use lucide-react icons via shadcn/ui).

2. **Posts list (`/posts`):**
   - List all posts sorted by date.
   - Each entry shows title, date, optional summary.

3. **Individual post (`/posts/:slug`):**
   - Render the MDX content with the PostLayout.
   - Show title, date, reading time (optional).

4. **Archives (`/archives`):**
   - Group posts by year.

5. **Dark mode:**
   - Default to dark (matching current site).
   - Use shadcn/ui's theme provider (`next-themes` pattern adapted for Vite).
   - Optional: add a toggle button.

6. **Navigation:**
   - Top nav bar with "posts" and "archives" links (matching current `menu.main` config).

### Phase 4: Static Site Generation

The site needs to be fully static (pre-rendered HTML) for GitHub Pages.

**Option A: vite-ssg (recommended for simplicity)**
```bash
pnpm add -D vite-ssg
```
Configure routes, and it pre-renders each route to static HTML at build time.

**Option B: Custom pre-render script**
Build the SPA with Vite, then use a headless browser (e.g., puppeteer) or `react-dom/server` to render each known route to HTML files.

**Option C: vike (formerly vite-plugin-ssr)**
More full-featured SSR/SSG framework. Heavier, but robust.

**Recommendation:** Start with Option A for simplicity. If it doesn't fit, fall back to Option B with a custom Node script using `react-dom/server`'s `renderToString`.

**Build output:**
```
dist/
├── index.html
├── posts/index.html
├── posts/chile-public-access-api/index.html
├── archives/index.html
├── CNAME
└── assets/
    ├── *.js
    └── *.css
```

### Phase 5: GitHub Actions Workflow

Replace the Hugo workflow with a pnpm-based one.

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: latest

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

**Note:** The `CNAME` file must be in `public/` so Vite copies it to `dist/` at build time.

### Phase 6: Verification & Cutover

1. **Local verification:**
   ```bash
   pnpm build
   pnpx serve dist    # preview the static output
   ```
   Walk through every page. Confirm:
   - [ ] Home page profile renders correctly
   - [ ] Social links work
   - [ ] Posts list shows all posts
   - [ ] Individual post content renders correctly (headings, links, lists)
   - [ ] Archives page groups by year
   - [ ] Dark mode is default
   - [ ] Navigation between pages works
   - [ ] No broken links or missing assets

2. **Optional: preview deploy**
   Push to a test branch and use GitHub Pages preview, or deploy to a temporary Netlify/Vercel site.

3. **Cutover:**
   ```bash
   git checkout main
   git merge migrate/vite-react
   git push origin main
   ```

4. **Post-cutover:**
   - Verify `self.rosebush.garden` serves the new site.
   - Delete the migration branch:
     ```bash
     git branch -d migrate/vite-react
     git push origin --delete migrate/vite-react
     ```

---

## Files to Delete (Hugo Cleanup)

These are removed during Phase 1 on the migration branch:

| File/Directory       | Reason                              |
|----------------------|-------------------------------------|
| `config.toml`        | Hugo config, replaced by Vite/React |
| `archetypes/`        | Hugo scaffolding                    |
| `themes/`            | PaperMod submodule                  |
| `.gitmodules`        | Submodule reference for PaperMod    |
| `.hugo_build.lock`   | Hugo lock file                      |
| `public/`            | Hugo build output (Vite uses `dist/`) |
| `.nojekyll`          | Move to `public/` for Vite to copy, or handle in CI |

## Files to Preserve / Migrate

| File                                | Action                              |
|-------------------------------------|-------------------------------------|
| `CNAME`                             | Move to `public/CNAME`             |
| `content/posts/chile-public-access-api.md` | Rename to `.mdx`, keep frontmatter |
| `content/archives.md`               | Replace with React Archives page    |
| `README.md`                         | Update to reflect new stack         |
| `.github/workflows/gh-pages.yml`    | Replace with new pnpm-based workflow |

---

## Key Dependencies

```json
{
  "dependencies": {
    "react": "^18.x",
    "react-dom": "^18.x",
    "react-router-dom": "^6.x"
  },
  "devDependencies": {
    "@mdx-js/react": "^3.x",
    "@mdx-js/rollup": "^3.x",
    "@tailwindcss/typography": "^0.5.x",
    "@tailwindcss/vite": "^4.x",
    "tailwindcss": "^4.x",
    "rehype-highlight": "^7.x",
    "remark-frontmatter": "^5.x",
    "remark-mdx-frontmatter": "^4.x",
    "typescript": "^5.x",
    "vite": "^6.x"
  }
}
```

shadcn/ui components (built on Base UI primitives) are installed on-demand via `pnpx shadcn@latest add <component>` and live in `src/components/ui/`.

---

## Risk Mitigation

| Risk                                    | Mitigation                                      |
|-----------------------------------------|-------------------------------------------------|
| Live site breaks during migration       | All work on separate branch; `main` untouched   |
| Content formatting differences          | Visually compare Hugo output vs new site         |
| SEO/URL changes break existing links    | Keep identical URL structure (`/posts/slug/`)    |
| CNAME lost during deploy                | Place `CNAME` in Vite's `public/` directory      |
| SSG doesn't generate all routes         | Explicitly declare all routes for pre-rendering  |
| Dark mode flash on load                 | Inline script in `index.html` to set theme early |

---

## Future Enhancements (Out of Scope)

These are not part of the migration but become easy to add afterward:

- RSS feed generation (build script)
- Search functionality
- Tags/categories pages
- Image optimization pipeline
- Reading time estimates
- Table of contents for posts
- MDX custom components (callouts, embedded demos, etc.)
