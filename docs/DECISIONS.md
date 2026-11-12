# Anivio — Architecture Decision Records & Conflict Register

> **Definitive Single Source of Truth for Architectural Decisions & Conflict Resolutions**  
> Status: **LOCKED BASELINE (Version 1.0)**  
> Last Updated: 12 November 2026

This document records the architectural constraints, locked engineering choices, open decisions, and formal resolutions to contradictions discovered between initial planning documents.

---

## 1. Conflict Resolutions Between Source Materials

During source analysis of the two original project documents (`Anime Platform Product Architecture Plan.docx` and `Anime Platform - Build Planning Document.docx`), four critical technical contradictions were identified. In accordance with the project charter, the **Build Planning Document** is adopted as the primary authoritative baseline. The resolutions are recorded below:

| # | Conflict Area | Architecture Plan Recommendation | Build Planning Baseline (LOCKED) | Formal Resolution & Rationale |
|---|---|---|---|---|
| **CR-01** | **Primary Anime Metadata Provider** | Jikan REST API v4 (Unofficial MyAnimeList API) | Tenrai API v1 as Primary, AniList as Secondary, PostgreSQL snapshots | **Resolved in favor of Tenrai + Snapshots**. Jikan v4 is officially unmaintained and suffers severe 429 rate limit bans. Tenrai provides modern MAL compatibility. Snapshots guarantee zero site downtime. |
| **CR-02** | **Next.js Caching Mechanism** | Next.js `cacheLife` directive around raw `fetch` calls | Three-layer caching: `'use cache'`, Redis token bucket, PostgreSQL snapshots | **Resolved in favor of Three-Layer Model**. `cacheLife` alone cannot buffer external rate-limited APIs across serverless instances. Multi-layer solves presentation, budget, and durability independently. |
| **CR-03** | **Comment Tree Maximum Nesting Depth** | Up to 10 levels deep | Hard-capped at 8 levels via database constraint | **Resolved in favor of 8 levels**. 10 levels produces severe horizontal clipping on 320px mobile viewports. Level 8 is enforced via PostgreSQL `CHECK (depth BETWEEN 1 AND 8)`. |
| **CR-04** | **Real-Time Communication & WebSockets** | Considered for real-time notification alerts | Strictly excluded from MVP scope | **Resolved in favor of Strict Exclusion**. Stateful WebSocket infrastructure dramatically increases serverless hosting costs. Asynchronous in-app polling is sufficient for V1. |

---

## 2. Locked Architecture Decision Records (ADRs)

### ADR-001: Framework Selection — Next.js 16 (App Router)
- **Status**: `LOCKED`
- **Context**: Anivio requires an image-heavy, highly readable frontend paired with server actions and edge caching, developed by a solo engineer.
- **Decision**: Adopt Next.js 16 with App Router, TypeScript, and React Server Components enabled.
- **Alternatives Rejected**:
  - *Separate Express / Fastify API + React SPA*: Requires maintaining two repositories, duplicate type declarations, and multiple deployments.
  - *Remix / React Router v7*: Smaller component ecosystem for headless anime cards.
- **Consequences**: Shipped client-side JavaScript remains under 180 KB; mutations occur cleanly via typed Server Actions.

---

### ADR-002: Persistence Engine — PostgreSQL 16+ on Neon
- **Status**: `LOCKED`
- **Context**: The platform's core differentiator is hierarchical discussion comments and relational social integrity (votes, memberships, watchlists).
- **Decision**: Use managed PostgreSQL 16+ on Neon with `ltree`, `pg_trgm`, and `citext` extensions.
- **Alternatives Rejected**:
  - *MongoDB*: Recursive tree traversal and vote deduplication require complex application-level locks.
  - *MySQL*: Lacks native hierarchical materialized path indexing (`ltree`).
- **Consequences**: Complex comment branches are retrieved in a single indexed query (<400ms server time).

---

### ADR-003: Hierarchical Comment Storage — PostgreSQL `ltree`
- **Status**: `LOCKED`
- **Context**: Threaded discussions can reach hundreds of nodes. Recursive Common Table Expressions (CTEs) degrade severely under load.
- **Decision**: Implement materialized path hierarchy using PostgreSQL `ltree` datatype, indexed via GiST. Comment insertions utilize sequence labels and advisory locks.
- **Alternatives Rejected**:
  - *Adjacency List (Parent Pointers only)*: Requires expensive recursive queries that lock database CPU during high-traffic episodes.
  - *Nested Sets*: Requires rewriting arbitrary left/right integers across thousands of rows on every single comment insertion.
- **Consequences**: Entire discussion trees are fetched in conversation order in one query: `WHERE path <@ root ORDER BY path`.

---

### ADR-004: Data Access Layer — Drizzle ORM
- **Status**: `LOCKED`
- **Context**: Need type-safe database queries without heavy serverless runtime overhead.
- **Decision**: Adopt Drizzle ORM with SQL migrations committed directly to source control.
- **Alternatives Rejected**:
  - *Prisma ORM*: Heavier Rust query engine increases serverless cold starts; awkward integration with native `ltree` syntax.
- **Consequences**: Zero runtime query engine overhead; transparent SQL generation; strict TypeScript type safety.

---

### ADR-005: Authentication Strategy — Auth.js with Database Sessions
- **Status**: `LOCKED`
- **Context**: Platform requires identity for tracking and posting, without password management liability.
- **Decision**: Deploy Auth.js (NextAuth v5) using Google and Discord OAuth 2.0 with database sessions stored in PostgreSQL.
- **Alternatives Rejected**:
  - *Custom JWT Cookies*: Revocation is difficult during moderation emergencies.
  - *Third-party SaaS (Clerk / Supabase)*: Introduces monthly costs and vendor lock-in.
- **Consequences**: Moderator can instantly invalidate malicious user sessions by deleting rows in the `sessions` table.

---

### ADR-006: External Anime Metadata Engine & Snapshot Durability
- **Status**: `LOCKED`
- **Context**: External anime APIs frequently suffer outages or rate limiting. Anivio must not crash when upstream APIs fail.
- **Decision**: Deploy Tenrai API v1 as primary adapter, AniList as secondary fallback, and store all fetched data in a local `provider_snapshots` table.
- **Alternatives Rejected**:
  - *Self-hosted Jikan*: High maintenance burden and proxy rotation costs.
  - *Direct MyAnimeList Scraper*: Fragile, subject to legal takedown.
- **Consequences**: Anivio renders uninterrupted from local snapshots with a `DegradedNotice` during external provider downtime.

---

### ADR-007: Algorithmic Ranking Strategy — Wilson Score & Hot Rank
- **Status**: `LOCKED`
- **Context**: Naive "upvotes minus downvotes" sorting allows single-vote comments to outrank well-supported discussions and buries new content.
- **Decision**: Implement the Wilson Score Interval lower bound ($z=1.96$) for "Best" sorting and logarithmic time-decay for "Hot" sorting inside PostgreSQL immutable functions.
- **Alternatives Rejected**:
  - *Raw Net Vote Score*: Buries new submissions; controversial posts rise inappropriately.
- **Consequences**: Statistically robust comment ordering processed asynchronously via background jobs.

---

### ADR-008: Discussion Content Authoring — Restricted Tiptap Schema
- **Status**: `LOCKED`
- **Context**: Posts require inline anime tagging and formatting without opening XSS attack surfaces.
- **Decision**: Implement Tiptap headless editor with a restricted JSON schema. Disallow raw HTML, script execution, and inline base64 images.
- **Alternatives Rejected**:
  - *Raw Markdown*: Difficult to render live interactive entity hover cards cleanly.
  - *Raw HTML ContentEditable*: Extreme XSS attack surface.
- **Consequences**: Post content serializes to safe structured JSON; rendered server-side as static, accessible HTML.

---

### ADR-009: Media Storage & CDN — Cloudflare R2 Direct Uploads
- **Status**: `LOCKED`
- **Context**: User post attachments require cost-effective, performant storage.
- **Decision**: Presigned direct uploads to Cloudflare R2 served from an isolated custom domain (`media.anivio.app`).
- **Alternatives Rejected**:
  - *Amazon S3*: High unpredictable egress bandwidth costs.
  - *Vercel Blob*: Higher storage tier fees.
- **Consequences**: Zero egress fees; file bytes bypass application servers entirely.

---

## 3. Open Decisions Register

The following items are unresolved implementation details. Each has an authoritative recommended baseline and an assigned deadline:

| Decision ID | Open Topic | Evaluated Options | Authoritative Recommendation | Decide By |
|---|---|---|---|---|
| **OD-01** | Character-Level Tagging in Post Editor | A) Include in V1<br>B) Defer to P2 post-launch | **Option B (Defer to P2)**. Anime-level tagging achieves 90% of product differentiator with significantly less editor complexity. | Milestone 6 |
| **OD-02** | Monetization Activation Date | A) Never<br>B) At Launch<br>C) Post-Launch at user threshold | **Option C (Post-Launch)**. Launch on Vercel Hobby; upgrade to Pro plan when affiliate links go live. | Milestone 9 |
| **OD-03** | Community Creation Eligibility | A) Any signed-in user<br>B) Accounts >3 days old, max 2/wk<br>C) Application-only | **Option B (Accounts > 3 days old)**. Neutralizes automated community spam without manual admin queues. | Milestone 4 |
| **OD-04** | Content Rating Policy | A) SFW strictly<br>B) Allow NSFW with age-gate | **Option A (SFW only at launch)**. Uses provider safe-for-work filters to simplify moderation and legal exposure. | Milestone 3 |
| **OD-05** | Spoiler Scope Model | A) Post-level boolean<br>B) Post-level with optional episode field | **Option B (Post-level with optional episode field)**. Directly matches the needs of weekly episodic discussion threads. | Milestone 5 |
| **OD-06** | Author Post Editing Window | A) Unlimited<br>B) 24 hours<br>C) Immutable (Never) | **Option B (24 hours)**. Prevents authors from rewriting historical debate contexts while permitting typo corrections. | Milestone 4 |
| **OD-07** | Vote Roster Visibility | A) Aggregate score only<br>B) Public list of voters | **Option A (Aggregate score only)**. Public voter rosters encourage vote brigading and user retaliation. | Milestone 5 |
| **OD-08** | Username Change Frequency | A) Never<br>B) Once every 30 days | **Option B (Once every 30 days, old name reserved 90 days)**. Prevents identity confusion and impersonation. | Milestone 2 |
| **OD-09** | Application Telemetry & Analytics | A) None<br>B) Cookie-free aggregate analytics | **Option B (Cookie-free aggregate)**. Preserves privacy and eliminates GDPR cookie consent banner overhead. | Milestone 8 |
| **OD-10** | Outbound Email Notifications | A) None in V1<br>B) Weekly digest<br>C) Immediate alerts | **Option A (None in V1)**. Avoids deliverability, DNS warming, and unsubscribe compliance burdens before achieving MAU. | Milestone 9 |
| **OD-11** | Synopsis Display Length on Show Page | A) Full unedited text<br>B) Short excerpt + external link | **Option B (Short excerpt + link)**. Respects MyAnimeList copyright terms and keeps show views compact. | Milestone 3 |
| **OD-12** | IP Address Logging & Retention | A) Hash and keep 30 days<br>B) Do not store | **Option A (Hash and keep 30 days)**. Enables abuse investigation without holding personally identifiable data indefinitely. | Milestone 7 |
| **OD-13** | Database Preview Branch Strategy | A) Shared staging DB<br>B) Neon branch per PR | **Option B (Neon branch per PR)**. Fully isolates schema migrations during preview deployments. | Milestone 1 |
| **OD-14** | Source Code Repository Visibility | A) Private permanently<br>B) Public after launch | **Option B (Private until launch, public post-launch)**. Serves as portfolio artifact once security audit is complete. | Milestone 9 |
| **OD-15** | Algorithmic Ranking Tie-Breaker | A) Entity UUID<br>B) Creation timestamp | **Option B (Creation timestamp)**. Resolves ties deterministically without jumping positions. | Milestone 5 |

