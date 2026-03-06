# Migration Progress

## Phase 1: Scaffold the New Project
- [x] Create `migrate/vite-react` branch
- [x] Remove Hugo artifacts (config.toml, archetypes/, themes/, .gitmodules, .hugo_build.lock, public/, .nojekyll)
- [x] Scaffold Vite + React + TypeScript (`pnpm create vite@latest . --template react-ts`)
- [x] Verify dev server runs (`pnpm dev`)
- [ ] Install Tailwind CSS (`pnpm add -D tailwindcss @tailwindcss/vite @tailwindcss/typography`)
- [ ] Configure Tailwind in `vite.config.ts` and `src/index.css`
- [ ] Install and initialize shadcn/ui (`pnpx shadcn@latest init`)
- [ ] Install React Router (`pnpm add react-router-dom`)
- [ ] Install MDX support (`pnpm add -D @mdx-js/rollup @mdx-js/react remark-frontmatter remark-mdx-frontmatter rehype-highlight`)
- [ ] Configure MDX plugin in `vite.config.ts`
- [ ] Move `CNAME` into `public/`
- [ ] Clean up Vite boilerplate (default App.tsx, App.css, assets)

## Phase 2: Content Pipeline
- [ ] Rename `chile-public-access-api.md` to `.mdx`
- [ ] Create `src/lib/content.ts` with `import.meta.glob` loader
- [ ] Verify frontmatter parsing works (title, date, draft)
- [ ] Test rendering a post's MDX content

## Phase 3: Build the Pages
- [ ] Set up React Router in `App.tsx` with routes: `/`, `/posts`, `/posts/:slug`, `/archives`
- [ ] Create `src/pages/Home.tsx` — profile section with name, subtitle, social links
- [ ] Create `src/pages/Posts.tsx` — list all posts sorted by date
- [ ] Create `src/pages/Post.tsx` — render individual MDX post
- [ ] Create `src/pages/Archives.tsx` — posts grouped by year
- [ ] Create navigation bar (posts, archives links)
- [ ] Set up dark mode as default

## Phase 4: Static Site Generation
- [ ] Choose SSG approach (vite-ssg, custom script, or vike)
- [ ] Configure pre-rendering for all routes
- [ ] Verify `pnpm build` produces correct `dist/` output with all HTML pages

## Phase 5: GitHub Actions Workflow
- [ ] Replace `.github/workflows/gh-pages.yml` with pnpm + Vite build workflow
- [ ] Verify CNAME is included in build output

## Phase 6: Verification & Cutover
- [ ] `pnpm build` + preview with `pnpx serve dist`
- [ ] Home page profile renders correctly
- [ ] Social links work
- [ ] Posts list shows all posts
- [ ] Individual post renders correctly
- [ ] Archives page groups by year
- [ ] Dark mode is default
- [ ] Navigation works
- [ ] No broken links or missing assets
- [ ] Merge `migrate/vite-react` into `main`
- [ ] Verify `self.rosebush.garden` serves the new site
- [ ] Delete migration branch
