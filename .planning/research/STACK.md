# Stack Research

**Domain:** Personal productivity Life OS (CLI + Obsidian + Web dashboard)
**Researched:** 2026-03-28
**Confidence:** HIGH

## System Topology

Life OS has three layers with distinct technology needs:

1. **CLI Layer** -- Claude Code skills (markdown prompt files). Already exists. No runtime code; Claude reads/writes vault files directly. Zero additional tech needed here.
2. **Vault Layer** -- Obsidian markdown files as the single source of truth. Skills read/write these directly. Needs conventions, not libraries.
3. **Web Dashboard Layer** -- Read-only status overview. This is the only layer that needs a traditional web stack.

The critical insight: the CLI and vault layers are already functional with zero dependencies. Stack decisions are almost entirely about the **web dashboard** and a thin **shared utilities** layer for any future automation scripts.

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended | Confidence |
|------------|---------|---------|-----------------|------------|
| Next.js | 15.5.x | Web dashboard framework | Server Components for fast initial load of vault data; API routes for future webhook integrations; largest React ecosystem for dashboard components. Already stable with Turbopack. | HIGH |
| React | 19.x | UI library | Ships with Next.js 15; Server Components enable reading vault files on server without client bundle overhead. | HIGH |
| TypeScript | 5.5+ | Type safety | Non-negotiable for a system with multiple data shapes (tasks, goals, CRM contacts, calendar events). Native Node.js support means scripts run with `node --strip-types`. | HIGH |
| Tailwind CSS | 4.2.x | Styling | CSS-native config (no JS config file), 5x faster builds, pairs perfectly with shadcn/ui. v4 is stable and production-ready. | HIGH |
| Node.js | 22 LTS | Runtime | Native TypeScript support (--strip-types, no flags needed since 22.18). LTS means stability for a personal system. | HIGH |

### Web Dashboard Libraries

| Library | Version | Purpose | Why Recommended | Confidence |
|---------|---------|---------|-----------------|------------|
| shadcn/ui | CLI v4 | Component library | Copy-paste components, not a dependency. Tailwind-native. Radix UI primitives. Perfect for dashboards with tables, cards, progress bars. | HIGH |
| Tremor | latest | Dashboard charts | Built on Recharts + Radix. Purpose-built for analytics dashboards. High-level API for goal progress charts, task velocity, CRM activity. Less code than raw Recharts for this use case. | MEDIUM |
| TanStack Query | 5.95.x | Client data fetching | Smart caching + background refetch for dashboard polling. Pairs with Server Components: server fetches initial data, TanStack handles client-side refresh. | HIGH |
| date-fns | 4.x | Date manipulation | Tree-shakable, functional API, excellent TypeScript types. Used heavily for daily/weekly note date calculations, calendar formatting, staleness checks. | HIGH |
| Zod | 4.3.x | Schema validation | Config validation (config.yaml shape), API response validation, type inference. 14x faster string parsing in v4, 2x smaller bundle. | HIGH |

### Vault Manipulation (shared utilities)

| Library | Version | Purpose | Why Recommended | Confidence |
|---------|---------|---------|-----------------|------------|
| gray-matter | 4.0.3 | Frontmatter parsing | Battle-tested YAML frontmatter parser. Used by Astro, Gatsby, Vitepress. Essential for reading/writing Obsidian note metadata. | HIGH |
| remark + unified | remark@15 | Markdown AST | Programmatic markdown manipulation when simple string ops are insufficient. Parse tasks from daily notes, extract sections, transform content. Only use when regex/string manipulation becomes fragile. | MEDIUM |
| yaml | 2.x | YAML parsing | Parse config.yaml and goals.yaml. Native YAML handling for vault config files. | HIGH |
| globby | 14.x | File globbing | Find notes by pattern across vault directories. ESM-native. | HIGH |

### Development Tools

| Tool | Purpose | Notes | Confidence |
|------|---------|-------|------------|
| Biome | Lint + format | Single binary, 50x faster than ESLint+Prettier. 423+ lint rules in v2.3. One config file. Use for the web dashboard codebase. | HIGH |
| Vitest | Testing | Vite-native, 4.1.x. Browser Mode now stable. Use for dashboard component tests and utility function tests. | HIGH |
| tsx | Script runner | For any one-off TypeScript scripts that need more than Node.js --strip-types supports (enums, decorators). 25x faster than ts-node. | MEDIUM |

### Infrastructure

| Technology | Purpose | Why Recommended | Confidence |
|------------|---------|-----------------|------------|
| Vercel | Dashboard hosting | Zero-config Next.js deployment. Free tier sufficient for personal dashboard. Git push to deploy. | HIGH |
| GitHub Actions | CI | Lint, test, deploy on push. Free for public/personal repos. | HIGH |
| Local filesystem | Data store | Obsidian vault IS the database. No external DB needed. Dashboard reads vault files via Node.js fs on the server. | HIGH |

## Architecture Decision: No Database

The vault is the database. The web dashboard reads markdown files from the local filesystem (or a synced copy) at request time via Next.js Server Components. This means:

- **No Postgres/SQLite/etc.** -- adding a database creates a sync problem with the vault
- **No ORM** -- no database means no ORM
- **Server Components read files directly** -- `fs.readFile` in server components, parsed with gray-matter + remark
- **If remote access needed later** -- expose vault via Obsidian REST API plugin or sync vault to server with Syncthing/iCloud

This is the correct architecture for a personal system where Obsidian is the source of truth. A database would duplicate data and create drift.

## Architecture Decision: Next.js over Alternatives

| Criterion | Next.js 15 | SvelteKit | Astro |
|-----------|-----------|-----------|-------|
| Server-side file reading | Server Components (native) | +server.ts (good) | Static build (mismatch) |
| Dashboard interactivity | Full React ecosystem | Great but smaller ecosystem | Islands -- too limiting |
| Component libraries | shadcn/ui, Tremor, hundreds more | Fewer options | Very few |
| Deployment | Vercel zero-config | Adapter-based | Adapter-based |
| Learning curve for maintainer | React is industry standard | Smaller community for help | Wrong tool for dashboards |

**Verdict:** Next.js wins because the dashboard needs interactivity (filtering, live refresh, drill-downs) AND server-side vault reading. SvelteKit is a credible alternative if you prefer Svelte, but the component ecosystem is significantly smaller for dashboard use cases. Astro is wrong for this -- it is content-site-first, not dashboard-first.

## Installation

```bash
# Initialize Next.js project (dashboard/)
npx create-next-app@latest dashboard --typescript --tailwind --app --src-dir

# Dashboard dependencies
cd dashboard
npm install @tanstack/react-query date-fns zod gray-matter yaml globby
npm install -D @biomejs/biome vitest @vitejs/plugin-react

# shadcn/ui (copy-paste, not a dependency)
npx shadcn@latest init
npx shadcn@latest add card table badge progress chart tabs

# Tremor charts
npm install @tremor/react
```

```bash
# Shared utilities (if extracted to a separate package later)
npm install gray-matter yaml date-fns zod globby
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Next.js 15 | SvelteKit | If you prefer Svelte and accept a smaller component ecosystem |
| Next.js 15 | Astro | Never for this project -- dashboards need interactivity, not islands |
| Tailwind CSS 4 | CSS Modules | If you strongly dislike utility classes (but shadcn/ui requires Tailwind) |
| Tremor | Recharts directly | If you need highly custom chart designs beyond Tremor's abstractions |
| Biome | ESLint + Prettier | If you need framework-specific plugins (eslint-plugin-next) that Biome lacks. Note: Next.js built-in linting handles most Next-specific rules. |
| date-fns | day.js | If bundle size is critical (2kB vs 12-40kB). For server-side dashboard, bundle size is irrelevant. |
| date-fns | Temporal API | When you can target only Chrome 144+/Firefox 139+. Temporal reached Stage 4 in March 2026 but Node.js support is still behind browsers. |
| Vercel | Coolify / self-hosted | If you want to self-host on a VPS instead of using Vercel's free tier |
| No database | SQLite | If vault-reading performance becomes a bottleneck with thousands of notes (unlikely for personal use) |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Prisma / Drizzle / any ORM | No database. Vault is the data store. Adding a DB creates sync drift. | Direct fs reads with gray-matter |
| MongoDB / Postgres / SQLite | Same reason. Obsidian vault is already a document store (markdown files). | Filesystem reads |
| Express / Fastify (standalone) | Next.js API routes cover any API needs. No reason for a separate server. | Next.js Route Handlers |
| Moment.js | Deprecated, 300kB+, mutable API. | date-fns |
| ts-node | 25x slower than tsx, heavy dependencies. | tsx or Node.js native --strip-types |
| Create React App | Deprecated. No SSR. | Next.js |
| ESLint + Prettier (separate) | Two tools, 127+ npm packages, 4 config files, 50x slower than Biome. | Biome |
| Electron / Tauri | Over-engineered for a personal dashboard. Web browser is the "app". | Next.js web app |
| Redux / Zustand | TanStack Query handles server state. Dashboard has minimal client state. | TanStack Query + React useState |
| Styled Components / Emotion | Runtime CSS-in-JS is dead in RSC world. Tailwind is compile-time. | Tailwind CSS |

## Stack Patterns by Variant

**If dashboard is local-only (runs on your machine):**
- Use `next dev` in development mode or `next build && next start`
- Read vault files directly from local filesystem
- No deployment needed, access at localhost:3000

**If dashboard needs remote access (phone, other devices):**
- Deploy to Vercel
- Sync vault to server via GitHub (commit vault) or Syncthing
- Or: use Obsidian REST API plugin as a data source instead of direct fs reads
- Add authentication (NextAuth.js or Clerk) since data is now exposed

**If you want to add automation scripts later (cron jobs, webhooks):**
- Write TypeScript scripts in a `scripts/` directory
- Run with `node --strip-types script.ts` (Node.js 22 LTS)
- Share vault-reading utilities with the dashboard via a `packages/shared/` workspace

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| Next.js 15.5.x | React 19.x | React 19 is required for Server Components |
| shadcn/ui CLI v4 | Tailwind CSS 4.x | shadcn/ui v4 supports both Tailwind v3 and v4 |
| Tremor | React 18/19, Tailwind 3/4 | Check Tremor docs for Tailwind v4 migration status |
| TanStack Query 5.x | React 18/19 | Works with Server Components via hydration |
| Biome 2.3.x | N/A | Standalone binary, no runtime dependencies |
| Vitest 4.1.x | Node.js 18+ | Uses Vite under the hood |

## Sources

- [Next.js 15.5 release](https://nextjs.org/blog/next-15-5) -- version and features verified
- [Tailwind CSS v4.2 release](https://tailwindcss.com/blog/tailwindcss-v4) -- CSS-native config confirmed
- [shadcn/ui CLI v4 changelog](https://ui.shadcn.com/docs/changelog/2026-03-cli-v4) -- March 2026 update
- [Zod v4 release notes](https://zod.dev/v4) -- performance improvements verified
- [TanStack Query docs](https://tanstack.com/query/latest) -- v5.95.x, RSC integration
- [Node.js TypeScript docs](https://nodejs.org/en/learn/typescript/run-natively) -- native --strip-types support
- [Biome migration guide](https://biomejs.dev/guides/migrate-eslint-prettier/) -- rule coverage and speed claims
- [Vitest 4.0 announcement](https://vitest.dev/blog/vitest-4) -- Browser Mode stable
- [Tremor dashboard components](https://www.tremor.so/) -- Recharts-based, dashboard-focused
- [gray-matter npm](https://www.npmjs.com/package/gray-matter) -- v4.0.3, frontmatter parsing
- [remark/unified](https://github.com/remarkjs/remark) -- markdown AST manipulation
- [Temporal API Stage 4](https://www.wearedevelopers.com/en/magazine/544/the-temporal-api-how-javascript-dates-might-actually-be-getting-fixed-544) -- March 2026, browser support only

---
*Stack research for: Life OS personal productivity system*
*Researched: 2026-03-28*
