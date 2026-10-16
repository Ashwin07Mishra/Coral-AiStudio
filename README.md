# Coral: Creative Intelligence Engine

AI-generated, brand-safe ad creatives for automotive campaigns, built at Innocean India for Hyundai.

> This repository holds the **project overview only**. The source code lives in a private repository.

An User Admin writes a campaign brief, Claude turns it into five creative concept directions with image prompts and copy, and an image model renders them. Brand managers approve or reject, and dealers download the approved creatives with an editable text overlay.

---

## Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15 (App Router), React 19, Tailwind v4, NextAuth v5 |
| Backend | Node.js 22, Express 5, TypeScript, BullMQ (Redis) |
| Database | PostgreSQL, Prisma 6 |
| AI | Claude API (concepts, copy, prompts), fal.ai Flux Pro (images) |
| Cloud | AWS (Mumbai region): EC2, RDS, S3, CloudFront |
| Tooling | npm workspaces monorepo, PM2, Playwright |

## Repository layout (private source)

```
apps/
  web/        Next.js: Ocean Admin, Brand and Dealer dashboards
  api/        Express API: auth, campaigns, AI jobs, S3
packages/
  db/         Prisma schema, client, seed
  shared/     Cross-package TypeScript types
```

---

## User roles

| Role | Capability |
|---|---|
| **Innocean Admin** | Creates campaigns, generates creatives, manages settings and brand guardrails |
| **Brand** | Reviews creatives, approves or rejects them, gives feedback |
| **Dealer** | Browses approved creatives, edits the text overlay, downloads |

Dealer accounts are created by the Ocean Admin.

---

## Generation flow

```
Brief (Admin) ──► Claude ──► 5 concept directions + image prompts + copy
                                       │
                                       ▼
                         fal.ai / Flux Pro: low-res drafts
                                       │
                           Approve in Ocean review
                                       │
                                       ▼
                         fal.ai / Flux Pro: high-res final
                                       │
                                       ▼
                         S3 + Postgres metadata
                                       │
                                       ▼
                    Brand approves ──► Dealer downloads
```

**Low-res first, high-res only on approval**, which keeps image-generation cost under control.

---

## Highlights

- **Role-based workflow** across three dashboards (admin, brand, dealer) with an activity log and notifications.
- **Background jobs** for generation through BullMQ and Redis, so long AI calls never block the API.
- **Brand guardrails** configured per brand, applied to every prompt.
- **Cost-aware rendering** with a cost preview before generation and a two-stage low-res to high-res pipeline.
- **Production operations:** PM2 process management, a scheduled retention job that prunes old activity logs and notifications, and a health endpoint for load-balancer checks.
- **End-to-end tests** with Playwright covering login, navigation, settings, the catalogue, the campaign wizard and concept filters. The tests never trigger paid AI generation, so CI doesn't burn credits.
- **Security-minded deployment:** environment checks at boot, secret rotation guidance, HTTPS and origin allow-lists, and client-IP handling designed to resist header spoofing.

---

## Local development (summary)

Requires Node 22+, PostgreSQL 16 and Redis.

```bash
npm install
cp .env.example .env     # set DATABASE_URL and AUTH_SECRET
npm run db:generate && npm run db:migrate && npm run db:seed
npm run dev              # web on :3000, api on :4000
```

`db:seed` creates a catalogue and one demo user per role for local use.

---

## Status

Built and run as an internal tool. Source is private because it is client work.
