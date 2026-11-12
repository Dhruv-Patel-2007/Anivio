# Anivio — Technology Stack Specification

> **Definitive Single Source of Truth for Architecture, Tooling & Dependency Choices**  
> Status: **LOCKED BASELINE (Version 1.0)**  
> Last Updated: 12 November 2026

This document specifies the exact technology choices, versions, architectural roles, alternatives evaluated, and operational constraints for the Anivio Anime Community Platform.

---

## 1. Stack Status Overview

| Technology | Category | Status | Target Version / Policy | Deployment Target |
|---|---|---|---|---|
| **Next.js** | Full-Stack Framework | **LOCKED** | `16.x` (App Router, Cache Components) | Vercel Serverless / Edge |
| **React** | UI Library | **LOCKED** | `19.x` (Server Components, Actions) | Browser & Server |
| **TypeScript** | Programming Language | **LOCKED** | `5.x` (Strict mode) | Build-time |
| **Tailwind CSS** | Styling Engine | **LOCKED** | `3.4.x` (CSS Variables tokens) | Client CSS bundle |
| **shadcn/ui** | Headless UI Primitives | **LOCKED** | Latest CLI (components copied to repo) | Client & Server |
| **PostgreSQL** | Relational Database | **LOCKED** | `16.x` (Neon Serverless) | Neon Managed Postgres |
| **Drizzle ORM** | Data Access Layer | **LOCKED** | `0.36.x` (`drizzle-kit 0.28.x`) | Server Runtime |
| **Auth.js** | Authentication & Sessions| **LOCKED** | `5.0.0-beta.x` (Database sessions) | Server Runtime |
| **Upstash Redis** | Cache & Rate Limiting | **LOCKED** | `@upstash/redis 1.34.x` | Upstash Serverless REST |
| **Cloudflare R2** | Object Storage | **LOCKED** | S3-compatible API (`@aws-sdk/client-s3`) | Cloudflare Edge |
| **Tenrai API v1** | Primary Anime Metadata | **LOCKED** | REST v1 | External API Proxy |
| **AniList API** | Secondary Metadata Fallback| **LOCKED**| GraphQL v2 | External API Proxy |
| **Zod** | Schema Validation | **LOCKED** | `3.23.x` | Client & Server Runtime |
| **Tiptap** | Rich-Text Post Editor | **LOCKED** | `2.x` (Restricted node schema) | Client Island & Server |
| **Vitest** | Unit & Integration Test | **LOCKED** | `2.x` | CI Pipeline (GitHub Actions)|
| **Playwright** | End-to-End Test | **LOCKED** | `1.48.x` | CI Pipeline (GitHub Actions)|
| **Sentry** | Error & Performance Monitor| **LOCKED**| `@sentry/nextjs 8.x` | Cloud Dashboard |
| **Vercel** | Hosting & Edge Delivery | **LOCKED** | Hobby (Launch) → Pro (Monetized) | Production Hosting |
| **GitHub Actions** | CI/CD Automation | **LOCKED** | Ubuntu Runners | Continuous Integration |

---

## 2. Deep Dive: Core Technologies

### Next.js 16 (App Router)
- **Status**: **LOCKED**
- **Purpose**: Full-stack framework coordinating React Server Components, Server Actions, Route Handlers, and Edge Caching.
- **Why Selected**: Combines frontend interface and server logic in a unified TypeScript modular monolith. Next.js 16's Cache Components (`'use cache'`) provides granular caching of upstream metadata.
- **Where Used**: Entire application layer (`src/app/`).
- **Alternatives Rejected**:
  - *Separate Express / Fastify API + React SPA*: Two repositories, duplicate type definitions, cold start delays, and two distinct deployments for a solo developer.
- **Limitations**: Vercel Hobby plan limits cron execution to once per day. Addressed by making background tasks event-driven or using GitHub Actions.

### PostgreSQL 16+ on Neon
- **Status**: **LOCKED**
- **Purpose**: Primary relational persistence engine.
- **Why Selected**:
  1. Native support for `ltree` extension, allowing single-query traversal of hierarchical comment trees without recursive CTE performance overhead.
  2. Built-in `pg_trgm` extension for fuzzy anime title search.
  3. `citext` extension for case-insensitive username and community slug uniqueness.
  4. Serverless branching on Neon enables isolated database environments for preview deployments.
- **Alternatives Rejected**:
  - *MongoDB*: Document model makes feed ranking, join tables (votes, memberships), and relational integrity complex to enforce in application code.
- **Limitations**: Max connection pool limits on serverless functions. Addressed using Neon's connection pooler string.

### Drizzle ORM
- **Status**: **LOCKED**
- **Purpose**: Type-safe relational data modeling, query builder, and SQL migration manager.
- **Why Selected**: Generates zero runtime overhead, writes transparent SQL without an abstracted engine, natively supports custom PostgreSQL types (`ltree`), and runs fast in serverless environments.
- **Alternatives Rejected**:
  - *Prisma ORM*: Heavier runtime query engine (WASM/Rust), higher serverless cold starts, and complex workarounds required to query `ltree` paths.
- **Limitations**: Requires developer familiarity with relational SQL syntax, which aligns with educational goals.

### Auth.js (NextAuth.js v5)
- **Status**: **LOCKED**
- **Purpose**: OAuth 2.0 provider integration and encrypted session management.
- **Why Selected**: Eliminates password storage liability. Native Drizzle adapter links OAuth profiles to PostgreSQL `accounts` and `sessions` tables. Database sessions allow instant moderation revocation.
- **Alternatives Rejected**:
  - *Custom JWT Authentication*: High risk of implementation errors (refresh token rotation, CSRF vulnerabilities).
  - *Clerk / Supabase Auth*: Adds third-party monthly recurring costs and vendor lock-in.

### Upstash Redis
- **Status**: **LOCKED**
- **Purpose**: Distributed rate limiting token bucket, single-flight cold cache mutex locks, and query response cache.
- **Why Selected**: Serverless HTTP/REST-based Redis client that does not exhaust connection limits across serverless instances.
- **Alternatives Rejected**:
  - *In-memory node-cache*: Resets on every serverless cold start; fails across multi-region instances.

### Cloudflare R2
- **Status**: **LOCKED**
- **Purpose**: Object storage for user-uploaded post media images.
- **Why Selected**: Zero egress fees, S3-compatible API, and direct presigned uploads keep user file payloads off the application server.
- **Alternatives Rejected**:
  - *AWS S3*: Unpredictable egress bandwidth charges.
  - *Vercel Blob*: Higher storage and transfer costs.

### External Anime Metadata Engine: Tenrai API (Primary) & AniList (Fallback)
- **Status**: **LOCKED**
- **Architecture**: Domain adapter pattern wrapping external calls with a 5-second timeout, 2 retries with jittered backoff, 429 response handling, single-flight locking, and automatic fallback to PostgreSQL `provider_snapshots`.
- **Reasoning**:
  - Jikan v4 is officially discontinued/unmaintained.
  - Tenrai API v1 is compatible with MyAnimeList IDs and is actively supported.
  - AniList GraphQL API serves as a secondary backup adapter if Tenrai experiences sustained downtime.
- **Alternatives Rejected**:
  - *Self-hosted Jikan*: Operational maintenance burden, proxy rotation costs, and ToS risks.
  - *Direct MyAnimeList Scraper*: Fragile, easily blocked, violates MAL Terms of Service.

### Tiptap Editor
- **Status**: **LOCKED**
- **Purpose**: Headless rich-text editor for discussion posts.
- **Why Selected**: Schema-driven architecture allows strict limitation of allowed HTML tags, preventing XSS injection while supporting custom nodes (like inline `@` anime tag cards). Serializes to clean JSON.
- **Alternatives Rejected**:
  - *Draft.js / Quill*: Outdated, complex SSR behavior.
  - *Raw Markdown Textarea*: Poor user experience for rendering live entity hover cards.

### Vitest & Playwright
- **Status**: **LOCKED**
- **Purpose**:
  - **Vitest**: Unit testing pure mathematical functions (Wilson Score, Hot Rank, path construction) and integration testing database constraints.
  - **Playwright**: End-to-end testing of 4 critical user journeys (Sign in, Post creation, Comment nesting, Watchlist toggle).
- **Alternatives Rejected**:
  - *Jest*: Slower execution, heavier configuration with TypeScript and ESM.
  - *Cypress*: Slower execution compared to Playwright in modern CI/CD pipelines.

---

## 3. Technology Evaluation & Decision Classifications

### Locked Technologies (Non-Negotiable)
- Next.js 16 (App Router)
- React 19
- TypeScript 5
- Tailwind CSS
- shadcn/ui
- PostgreSQL 16 + `ltree`, `pg_trgm`, `citext`
- Drizzle ORM
- Auth.js with Google & Discord OAuth
- Upstash Redis
- Cloudflare R2
- Tenrai API v1 + AniList Fallback + PostgreSQL Snapshots
- Tiptap
- Zod
- Vitest + Playwright
- Vercel (Hobby)

### Optional Technologies (To Be Evaluated Post-Launch)
- `@tanstack/react-virtual`: Virtualized rendering for comment threads exceeding 5,000 nodes.
- `sharp`: Server-side image optimization pipeline if client-side Canvas WebP encoding proves insufficient.

### Future Technologies (Planned for Post-MVP Monetization)
- Stripe / LemonSqueezy: Payment processing for Phase 3 cosmetic subscriptions.
- Tokyo Otaku Mode / Crunchyroll Affiliate SDKs: Contextual affiliate product integrations.

### Technologies Considered and Rejected
- **Prisma ORM**: Excessive serverless bundle size, Rust engine overhead, poor `ltree` support.
- **MongoDB**: Unsuited for deeply nested hierarchical queries and strict relational joins.
- **Jikan API v4**: Discontinued/unstable service.
- **Custom JWT Auth**: Unnecessary security overhead compared to database-backed OAuth sessions.
- **Algolia / Typesense**: Avoidable external infrastructure cost; PostgreSQL `tsvector` handles MVP volume cleanly.
- **WebSockets / Socket.io**: Unnecessary server overhead for asynchronous threaded discussions.

