# Comprehensive Prompt for AI: **Hikka UA Rebirth**

## Role & Mission
You are a **principal-level full-stack architect and implementation agent**. Build a complete, production-ready web platform from scratch that faithfully replicates the **Hikka UA** anime streaming experience.

This is **not** a generic streaming site. It must be a localized, high-performance, community-centric anime hub with:
- Ukrainian-first UX and language.
- Dark glassmorphism visual identity.
- Adaptive streaming via HLS/DASH.
- SEO-first architecture.
- Strong social/community interactions.

Deliver a complete codebase, infrastructure plan, DB schema, APIs, and setup docs.

---

## 1) Mandatory Technical Stack

### Core Frontend
- **Framework**: Next.js 14+ (App Router) — mandatory.
- **Language**: strict TypeScript (`"strict": true`) across app, API, schemas, components.
- **Styling**: Tailwind CSS with tokenized design system.
- **Client state**: Zustand (player state, UI modals, filters).
- **Server state**: TanStack Query (React Query) for API caching and revalidation.

### Backend & Data
- **Database**: PostgreSQL.
- **ORM**: Prisma (or Drizzle) with typed models and migrations.
- **Auth**: NextAuth.js (credentials + Google; optional Telegram/Discord).
- **Background jobs / sync**: scheduled ETL workers for metadata sync.

### Streaming
- **Player**: HLS.js and/or DASH.js.
- Do **not** rely on basic static `<video src="...">` logic.
- Must support adaptive bitrate playback and source failover.

### DevOps / Deployment
- Primary target: **Vercel**.
- Alternative: Dockerized deployment for DigitalOcean/Hetzner + CDN.
- Include CI/CD (lint, typecheck, test, build).
- Add a zero-cost deployment path for MVP rollout using free tiers (see section **10) Free Hosting Deployment Blueprint**).

---

## 2) Visual & UX Design System (Hikka Aesthetic)

### Global Theme
- **Dark mode only**.
- Cinematic, high-contrast, minimal clutter.

### Color Tokens
- `--bg-root`: `#000000` (or `#08080A` in layered sections).
- `--text-primary`: `#FFFFFF`.
- `--text-muted`: `#A0A0A0`.
- `--border-muted`: `#404040` or white alpha equivalent.
- `--accent`: vibrant cyan/blue (reserved for active/critical highlights).

### Glassmorphism Rules
Apply to cards, list rows, overlays:
- Semi-transparent dark surfaces (`bg-white/5` or `bg-gray-900/40`).
- Subtle border (`border-white/10`).
- Backdrop blur (`backdrop-blur-md` or `backdrop-blur-lg`).
- Rounded corners (`rounded-2xl` containers, `rounded-lg` controls).

### Typography
- Primary font: Inter / Montserrat / Geologica with robust Ukrainian Cyrillic coverage.
- Heading weights: 700+.
- Body weights: 400–500.
- Body line-height: `leading-relaxed`.

### Layout Spacing
- Wide, breathable layouts.
- Prefer `p-6`, `p-8`, `gap-8`, `gap-12`.
- Minimal visual noise, clear hierarchy.

---

## 3) Required Route & Component Architecture

### 3.1 Root Layout (`app/layout.tsx`)
- Sticky/fixed top header (dark/transparent).
- Left: minimal logo.
- Center nav: **Головна, Каталог, Календар, Топ, Блог**.
- Right controls: search, notifications, auth avatar/login.
- Mobile: replace center header nav with bottom navigation bar.
- Footer: **Про нас, DMCA, Соцмережі, Політика конфіденційності** + localized copyright.

### 3.2 Homepage (`app/page.tsx`)
- Hero carousel with high-res backdrop and anime focus info:
  - Title/logo.
  - Year, studio, score metadata.
  - Primary CTA: **Дивитись**.
- Horizontal category rails:
  - **Зараз дивляться**.
  - **Останні оновлення**.
  - **Новинки сезону**.
- Card content:
  - Poster.
  - Ukrainian title.
  - Type (TV/Movie).
  - Episode count badge (`rounded-full px-2 py-0.5`).

### 3.3 Anime Detail (`app/watch/[slug]/page.tsx`)
- Blurred poster as page background with smooth black fade.
- Left column:
  - Vertical poster.
  - Metadata block (Type, Episodes, Studio, Genre, Year).
  - Rating widget.
  - Share widget (**Поділитись**).
- Right column:
  - UA title + secondary EN/Romaji title.
  - **Про що?** synopsis.
  - Streaming/episodes module.

#### Episode/Streaming Module (Critical)
Use tabbed UI:
1. **Серії**
2. **Завантажити**
3. **Плеєр**

`Серії` tab requirements:
- Responsive episode card grid.
- Card includes: episode number, UA title, watched %, optional thumbnail.
- Progress bar for watched status.

`Плеєр` tab requirements:
- Custom player wrapper.
- Source selector (UA #1, UA #2...).
- Subtitle track selector.

### 3.4 Catalog (`app/catalog/page.tsx`)
- Desktop: persistent vertical filter panel.
- Mobile: collapsible filter sheet/dropdown.
- Filters:
  - Genre.
  - Year range.
  - Status (Ongoing/Completed).
  - Type (TV/Movie).
  - Studio.
  - Localization (Voiceover/Subtitles).
- Grid: `grid-cols-2` mobile, `grid-cols-4`/`grid-cols-5` desktop.

### 3.5 Community Module (below player)
- Threaded comments with nested replies.
- Upvote/downvote karma.
- User avatars + role flairs (e.g., **Куратор**, **Старожил**).
- UA-friendly formatting toolbar.
- Mandatory spoiler tag support.
- Real-time updates for new comments/replies (WebSocket or SSE/Pusher-compatible pattern).

### 3.6 Interactive Client Logic (Zustand + Framer Motion)
- **Global Player Store (Zustand)**:
  - Keep current playback time, duration, active episode id, and active source.
  - Reflect watched progress in episode cards in near real-time.
- **Page Transitions (Framer Motion)**:
  - Implement shared-element hero animation from anime card poster to detail-page poster.
  - Preserve smooth perception during route transitions.
- **Auto-Play Next Episode**:
  - On `ended` event, load next episode source automatically.
  - Update URL and page metadata client-side without full reload.

---

## 4) Backend/API Functional Requirements

### 4.1 Adaptive Streaming Logic
Implement a dedicated player controller that:
- Initializes HLS/DASH manifest playback.
- Detects bandwidth/quality metrics.
- Dynamically shifts between 1080p/720p/480p with minimal buffering.

### 4.2 Cross-Source Fallback (Balancer)
Player must:
1. Try primary source URL.
2. If playback init fails within **10 seconds** (404, CORS, network, fatal media error), auto-switch to next source.
3. Continue fallback chain until playable source is found.
4. Surface graceful UI feedback without hard page failure.

### 4.3 Watch History & Progress API
Create `/api/history` routes for:
- Upserting progress (`currentTime`, `duration`, `status: playing|completed`).
- Mapping user x anime x episode records.
- Returning watched % for episode list rendering.

### 4.4 Metadata Synchronization (ETL)
Implement scheduled sync from MAL/AniList:
- Pull synopsis, genres, cast, artwork, airing data.
- Prioritize and preserve Ukrainian-localized fields in storage.
- Resolve title aliases and deduplicate entries.

### 4.5 Database Schema Blueprint (Prisma + PostgreSQL)
Generate an explicit relational Prisma schema optimized for high-traffic reads:

- `User`
  - `id`, `username`, `email`, `avatar_url`, `role` (`ADMIN | MODERATOR | USER`), `karma_points`.
  - Relations to `WatchHistory`, `FavoriteAnime`, `Comment`, and user list entries.
- `Anime`
  - `id`, `slug` (URL-safe unique), `title_ua`, `title_en`, `synopsis_ua`, `poster_url`, `banner_url`, `status` (`ONGOING | COMPLETED`), `year`, `rating`.
  - Relations to `Episode`, `Genre`, `Studio`, metadata, and comments.
- `Episode`
  - `id`, `anime_id`, `episode_number`, `title_ua`, `video_sources` (JSON array, encrypted HLS/source objects), `torrent_link`.
  - Unique composite index on `(anime_id, episode_number)`.
- `WatchHistory`
  - `id`, `user_id`, `episode_id`, `progress_seconds`, `is_completed`, timestamps.
  - Unique composite index on `(user_id, episode_id)`.

Require efficient indexes for:
- `anime.slug`
- `episode.anime_id`
- `watch_history.user_id`
- `watch_history.episode_id`

### 4.6 Smart Parser & Balancer API (Serverless)
Create dedicated Next.js API routes (or route handlers) for source orchestration:

1. **Source Aggregator**
   - Returns normalized stream source array:
     - `[{ label: "UA #1", url: "...", type: "hls" }, { label: "UA #2", url: "...", type: "iframe" }]`.
   - Sources can be assembled from internal DB and/or trusted upstream APIs.
2. **Failure Detection / Health Check**
   - Before returning a source, verify availability:
     - For HLS, perform a manifest check against `.m3u8` endpoint.
     - Reject or deprioritize sources returning `403`, `404`, timeout, or invalid manifest body.
3. **Torrent Generator / Proxy Endpoint**
   - Serve `.torrent` via controlled API endpoint.
   - If upstream file is external, proxy stream download to hide origin and reduce hotlinking.
   - Add signed/expiring token support to protect torrent access URLs.

### 4.7 API Protection and Automation
- Apply rate limiting middleware on sensitive routes (`/api/stream/*`, `/api/torrent/*`, `/api/history/*`) to reduce scraping/abuse.
- Add audit logging for stream-source failures and fallback events.
- Enforce schema-based input validation (Zod/Valibot) on all write endpoints.

---

## 5) SEO, Performance, Accessibility, and Quality Gates

### SEO
- Fully rendered metadata on anime pages:
  - `<title>`.
  - `<meta name="description">`.
  - OpenGraph/Twitter tags.
  - JSON-LD structured data (`VideoObject`) including episode and embed metadata.

### Performance (target Lighthouse 100-ish envelope)
- `next/image` with AVIF/WebP.
- Lazy loading and dynamic imports (`next/dynamic`) for heavy client modules (player/comments editor).
- CDN caching strategy for images/static assets/manifests where safe.
- Code splitting and minimized hydration cost.
- External poster image proxy/loader must normalize and optimize formats (prefer `.webp`) and responsive sizes for mobile and desktop.

### Accessibility (a11y)
- Full keyboard navigation for player controls and episode lists.
- Semantic landmarks and ARIA labels.
- Visible focus states.
- High contrast compliance in dark mode.

### Security & Moderation
- Input validation and output sanitization on comments.
- CSRF/session protections via auth framework defaults.
- Basic moderation utilities (report/hide/spoiler enforcement).
- Rate limiting and anti-scraping controls for stream/torrent endpoints are mandatory.

### SEO Automation
- Auto-generate `sitemap.xml` and `robots.txt` from published anime slugs in the database.
- Revalidate sitemap when new anime entries or slug updates are synced.

---

## 6) Data Models (minimum)
Define typed models/entities for:
- User
- Profile
- Anime
- AnimeTitle (ua/en/romaji variants)
- Genre
- Studio
- Season
- Episode
- StreamSource
- SubtitleTrack
- WatchHistory
- Comment
- CommentVote
- Notification
- Watchlist

Ensure normalized relations, useful indexes, and audit fields (`createdAt`, `updatedAt`).

### Mandatory Personal Library States
Add user list categories and syncing logic for:
- `PLANNED`
- `WATCHING`
- `DROPPED`

These states must sync across devices and be queryable for profile dashboards.

---

## 7) Delivery Artifacts (must output)
1. Full project structure (folders/files).
2. DB schema + migrations.
3. Seed script with sample anime/episodes/comments.
4. API route handlers and validation schemas.
5. Reusable UI component library for Hikka style.
6. Player module with ABR + fallback.
7. Auth flows + protected routes.
8. Metadata generation utilities (SEO).
9. ETL/sync worker scaffold.
10. Setup docs:
    - `.env.example`
    - local development
    - build/deploy instructions
    - CI pipeline overview
11. Prisma schema and migration set containing `User`, `Anime`, `Episode`, and `WatchHistory` as explicit core tables.
12. Smart Parser/Balancer API handlers with manifest health-check and torrent proxy.
13. Zustand store definitions for player time/progress sync and episode autoplay-next state.
14. Framer Motion shared-element transition scaffold for card-to-detail hero animation.
15. Real-time comment module scaffold with spoiler-tag parsing (`[spoiler]...[/spoiler]`).

---

## 8) Definition of Done
The implementation is complete only if:
- Ukrainian-first UI is applied consistently.
- Hikka-like dark glassmorphism is visually coherent across all major pages.
- Anime detail page + player tabs + episode progress all work end-to-end.
- Streaming module performs adaptive playback and automatic fallback.
- Smart Parser verifies source health before client delivery and supports protected torrent proxy access.
- Comments system supports nesting, karma, and spoilers.
- Spoiler blocks in comments are blurred by default and reveal on explicit click.
- Auto-play next episode updates URL and metadata without full page reload.
- Zustand store reflects real-time progress updates in episode list while video is playing.
- SEO metadata and JSON-LD are verifiably rendered server-side.
- `sitemap.xml` and `robots.txt` are generated from live slug data.
- Typecheck/lint/tests/build all pass.

---

## 9) Constraints to Respect
- Avoid monolithic CSS; enforce Tailwind-based token patterns.
- Avoid untyped data flow (`any`) except narrowly justified adapters.
- Prefer SSR/SSG where it improves SEO/perceived performance.
- Keep client bundle size lean (defer heavy components).
- Maintain localization readiness for future UA/EN expansion.

---

## 10) Free Hosting Deployment Blueprint (MVP)
Prepare a deploy-ready configuration for free-tier infrastructure:

### Option A (Recommended): Vercel + Neon + Upstash
- **Frontend/API Runtime**: Vercel Hobby plan (Next.js native support).
- **PostgreSQL**: Neon free tier (serverless Postgres).
- **Rate Limiting / Cache**: Upstash Redis free tier.
- **Object/File Storage (optional)**: Cloudflare R2 free allowance or Supabase Storage free tier.

Required setup deliverables:
1. `vercel.json` with secure headers and route behavior for API handlers.
2. `.env.example` including:
   - `DATABASE_URL`
   - `NEXTAUTH_SECRET`
   - `NEXTAUTH_URL`
   - OAuth provider keys
   - `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN`
3. migration + seed commands documented for Neon connection.
4. deployment checklist for Vercel project linking and env var injection.

### Option B: Cloudflare Pages + Supabase
- If avoiding Vercel limits, provide compatibility notes for Cloudflare Pages/Workers runtime.
- Use Supabase Postgres + Auth adapters if needed, but preserve app-level auth UX and roles.

### CI/CD (GitHub Actions)
Include a workflow that runs on pull requests and main branch:
- `pnpm install --frozen-lockfile`
- `pnpm lint`
- `pnpm typecheck`
- `pnpm test` (if tests exist)
- `pnpm build`

### Cost & Safety Guardrails
- Keep services within free quotas by default:
  - aggressive image optimization/caching,
  - API route rate limits,
  - on-demand revalidation instead of excessive cron polling.
- Add fallback messaging when free-tier limits are hit (graceful degraded mode, not blank failure).
