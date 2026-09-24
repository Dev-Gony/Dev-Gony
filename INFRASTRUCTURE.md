# Infrastructure

> Last updated: 2026-09-24  
> Purpose: Keep hosting, database, auth, scheduled jobs, and deployment choices consistent across Dev-Gony projects.

## Current Infrastructure

| Project | Hosting / Runtime | Database / Auth | Scheduled Jobs | Status / Policy |
| --- | --- | --- | --- | --- |
| Algo-Memory | GitHub Pages | Neon Auth + Data API | GitHub Actions for build/deploy | Production uses Neon. Supabase is paused rollback only. |
| TechNews | GitHub Pages | No dedicated DB | GitHub Actions | Keep as static web + scheduled automation. |
| Re:Place | Vercel-compatible Next.js host | Neon Postgres | GitHub Actions every 6 hours | Production DB is Neon. Use `DATABASE_URL` only. |
| its-me | Vercel-compatible Next.js host | Supabase Auth + Postgres + Storage | CI / deployment smoke workflow | Keep on Supabase because Auth, SSR, RLS, and private Storage are tightly integrated. |
| Prism | Existing deployment | Supabase | Existing workflow | Do not change infrastructure while active development is in progress. |
| AutoShorts | Local | None | CI only | Keep local until heavy video rendering is separated from the web UI. |
| Career Agent | Local + Slack Socket Mode | Local/private data | Manual / local process | Do not deploy a permanent server yet. |
| gony-labs | GitHub only | Lab-specific local/container DBs | GitHub Actions | Portfolio/evaluation repository. No production deployment required. |
| Finance-News | Not decided | Not decided | Not decided | Define architecture before deployment. |
| NoticeGuard | Not deployed | Not decided | Not decided | Repository is not yet an operational service. |
| S.T.A | GitHub only | None | None | Deployment not required. |

## Default Infrastructure Policy

### Static frontend

Use:

```text
GitHub Pages
```

Good fit:
- static HTML/JS
- generated newspaper/archive pages
- no server-side API requirement

Current examples:
- Algo-Memory
- TechNews

### Next.js server application

Use:

```text
Vercel
```

Use this when the project needs:
- Next.js Route Handlers
- SSR / Server Components
- server-side environment variables
- dynamic backend requests

Do not move a working static site to Vercel only for convenience.

### New PostgreSQL database

Default:

```text
Neon first
```

Prefer Neon when the application mainly needs:
- PostgreSQL
- server-side SQL
- pooled connections
- scheduled collectors
- lightweight Data API / Auth integration

Current examples:
- Re:Place
- Algo-Memory

### Supabase

Use Supabase only when its integrated platform features materially reduce complexity.

Good reasons:
- Auth is tightly coupled to DB policies
- Supabase Storage is required
- browser-side Supabase SDK is already deeply integrated
- RLS and user-owned data are central to the application

Current examples:
- its-me
- Prism

Do not use a Supabase project just because a project needs PostgreSQL.

## Supabase Slot Policy

Free active project slots should be reserved for applications that actually need Supabase-specific features.

Current intended active projects:

```text
Prism
its-me
```

Paused / inactive legacy projects are not part of the production path.

```text
Algo-Memory Supabase -> paused rollback only
Re:Place Supabase    -> inactive legacy project
```

## Scheduled Jobs

Default:

```text
GitHub Actions
```

Use it for:
- periodic crawlers
- daily/weekly content generation
- static site publishing
- lightweight scheduled automation

Avoid waking a production database from CI when a change is unrelated to the database.

Example:
- Re:Place Neon DB checks run only for DB/crawler-related changes.
- Documentation-only changes should not trigger expensive full CI where avoidable.

## Heavy Processing

Do not force CPU-heavy workloads into a cheap web host.

Examples:
- video rendering
- FFmpeg
- long-running TTS/media pipelines
- large background jobs

Current policy for AutoShorts:

```text
Local MVP first
-> separate worker architecture later
-> deploy only after runtime/cost model is clear
```

## Secrets Policy

Never commit:
- PostgreSQL connection strings
- service-role keys
- API secrets
- OAuth client secrets
- LLM API keys

Use:
- GitHub Actions Secrets for scheduled jobs
- hosting-provider environment variables for deployed apps
- local `.env` files for local development

Browser-safe publishable keys may be exposed only when the provider explicitly designs them for public clients and authorization is enforced with RLS or equivalent server-side policy.

## Project Creation Checklist

Before starting a new service:

```text
1. Is a server actually required?
2. Can the frontend be static?
3. Does the app need Auth?
4. Does it need private file Storage?
5. Does it only need PostgreSQL?
6. Does it need scheduled jobs?
7. Does it perform heavy CPU/media processing?
```

Then choose:

```text
Static frontend
-> GitHub Pages

Next.js server app
-> Vercel

PostgreSQL only
-> Neon

Auth + Storage + deeply integrated RLS
-> Supabase

Scheduled batch
-> GitHub Actions

Heavy processing
-> Local / dedicated worker architecture
```

## Cleanup Rules

When migrating infrastructure:

1. Verify the new production path.
2. Keep the previous backend temporarily as rollback.
3. Remove old deployment configuration from the repository.
4. Remove unused secrets from GitHub / hosting providers.
5. Update README and this file.
6. Only then pause or delete the legacy service.

Do not keep duplicate production deployment paths without a rollback reason.
