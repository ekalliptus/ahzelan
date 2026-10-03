# Ahzelan.com

> Personal brand & digital-marketing site for **Ahzelan**: built with Astro, Bun, Tailwind v4, shadcn/ui, Supabase, and deployed on Cloudflare.

## Stack

| Layer        | Tech                                             |
| ------------ | ------------------------------------------------ |
| Framework    | **Astro 5** (SSR via Cloudflare adapter)         |
| Runtime      | **Bun 1.3** (install / dev / build)              |
| Styling      | **Tailwind CSS v4** (CSS-first `@theme` tokens)  |
| Components   | **shadcn/ui** (Radix + CVA), rethemed to brand   |
| Icons        | **lucide-react** (subset vendored as data)       |
| Animations   | `tailwindcss-animate` + custom CSS keyframes     |
| Data         | **Supabase** (Postgres, Storage, Auth, RLS)      |
| Deploy       | **Cloudflare Pages** (adapter: `@astrojs/cloudflare`) |
| Skeleton / lazy-load | shadcn `Skeleton` + `client:visible` + `loading="lazy"` |

## Brand (from the design system)

- **Primary**: Royal Blue `#2E4191` (sampled from the real logo)
- **Secondary**: Teal `#06b6d4` (gradient pair: blue → cyan)
- **Accent**: Amber `#f59e0b`
- **Success**: Green `#10b981` (status only)
- **Voice**: santai, friendly, sedikit playful, percaya diri, tetap profesional. "kamu" for reader, "aku" for Ahzelan. Primary CTA: **Chat Ahzelan** (WhatsApp).

All tokens live in [`src/styles/globals.css`](src/styles/globals.css) inside the `@theme` block.

## Project layout

```
src/
  components/
    ui/        shadcn primitives (Button, Card, Sheet, Accordion, Tabs, ...)
    layout/    Navbar, Footer, WhatsAppFloat, ScrollReveal
    sections/  Hero, TrustStrip, Services, LandingHighlight, Process,
               Recommendations, Articles, Testimonials, FAQ, FinalCTA
    public/    FaqAccordion, FaqWithSearch, RekomendasiFilter, GaleriGrid
    admin/     AdminShell, StatCard, DataTable, EntityForm, EntityManager,
               ConfirmDialog, ThemeManager, MediaLibrary, ArticleEditor,
               SettingsForm, DashboardOverview
  layouts/     PublicLayout, AdminLayout
  pages/       /, /tentang, /layanan, /jasa-pembuatan-landing-page,
               /rekomendasi, /faq, /blog, /blog/[slug], /galeri,
               /kebijakan-privasi, /bio-link, /admin/**, /api/admin/**
  lib/         supabase, auth, db, fallback, sanitize, utils, site, types
  middleware.ts (auth gate for /admin)
  styles/globals.css
supabase/
  migrations/0001_init.sql   schema + RLS + storage policies
public/                       brand assets, _headers, _redirects
```

## Local development

```bash
bun install
cp .env.example .env       # fill in PUBLIC_SUPABASE_URL + keys (optional)
bun run dev                # http://localhost:4321
```

Without env vars, the site renders from [`src/lib/fallback.ts`](src/lib/fallback.ts): real ahzelan.com content scraped and brand-voice placeholder for what's not public (pricing, FAQ detail, privacy).

### Supabase setup (one-time)

1. Create a project at https://supabase.com/dashboard
2. SQL Editor → paste & run `supabase/migrations/0001_init.sql`
3. Storage → create bucket `media` (public, 10MB max)
4. Project Settings → API → copy `URL` + `anon public` + `service_role secret`
5. Auth → add your admin email; insert it into `site_settings.whitelist_admins` jsonb
6. Set in `.env`:
   ```env
   PUBLIC_SUPABASE_URL=...
   PUBLIC_SUPABASE_ANON_KEY=...
   SUPABASE_SERVICE_ROLE_KEY=...
   ```
7. Optionally `psql` or seed the first article via `/admin/articles` once logged in.

## Build & deploy

```bash
bun run build              # produces ./dist
bun run preview            # local SSR preview
```

### Deploy to Cloudflare Pages

```bash
bunx wrangler pages deploy ./dist --project-name ahzelan
```

Or connect the repo to Cloudflare Pages dashboard with:
- Build command: `bun run build`
- Build output: `dist`
- Environment variables: same as `.env`

Set Supabase secrets in Pages → Settings → Environment Variables (mark `SUPABASE_SERVICE_ROLE_KEY` as secret).

## Admin

`/admin` (or `/admin/login`). All write operations go through `/api/admin/*` endpoints that:

1. Verify the session is a Supabase-authenticated user
2. Check the email is in `site_settings.whitelist_admins` (jsonb)
3. Use the service-role client (RLS bypass only after auth)

Public traffic only sees `status='published'` / `is_visible=true` rows (enforced by RLS in `0001_init.sql`).

## Content editing

Every entity is editable from `/admin`:

- **Dashboard**: counts + quick actions + recent activity
- **Halaman**: page metadata + SEO
- **Media**: upload to Supabase Storage, alt text, copy URL
- **Tema**: primary swatch, radius, font, live preview → saves to `site_settings`
- **Navigasi**: menu items + sort
- **Layanan**: services
- **Paket Harga**: landing page packages (featured = highlighted)
- **Testimoni**: quote + rating
- **FAQ**: accordion items
- **Rekomendasi**: tools/products with category filter
- **Artikel**: full editor (title, slug, HTML content, SEO, draft/publish, preview)
- **Pengaturan**: site name, contact, socials, default SEO, analytics script

## Security notes

- All admin writes require auth + email whitelist
- Article HTML is sanitized through an allowlist sanitizer (`src/lib/sanitize.ts`) on read before being rendered to the public
- CSP, HSTS, frame denial, no-sniff headers set in `public/_headers`
- Service role key never exposed to client; only used server-side in `/api/admin/*`
- Admin routes are `noindex` + `no-store` cached
