# Anivio

> **A community-first social discovery platform for anime and the wider
> world of Japanese and Asian fiction.**\
> **Project status:** Architecture baseline --- Version 1.1
> (consolidated)\
> **Documentation date:** 12 November 2026

Anivio brings fandom discussions, structured title information, personal
tracking, and community discovery into one place. Instead of treating an
anime or manga title as plain text inside a post, Anivio makes titles
first-class entities that users can explore, follow, discuss, and track
without losing the conversation.

The platform combines the community depth and discussion formats of
Reddit with the identity, profiles, discovery, and following mechanics
of a social network.

------------------------------------------------------------------------

## Contents

-   [1. Vision](#1-vision)
-   [2. Product Principles](#2-product-principles)
-   [3. Audience and Launch Strategy](#3-audience-and-launch-strategy)
-   [4. Core Features](#4-core-features)
-   [5. MVP Scope](#5-mvp-scope)
-   [6. Technology Stack](#6-technology-stack)
-   [7. System Architecture](#7-system-architecture)
-   [8. Core Domain Model](#8-core-domain-model)
-   [9. Feed and Ranking](#9-feed-and-ranking)
-   [10. Spoiler Protection](#10-spoiler-protection)
-   [11. Security and Moderation](#11-security-and-moderation)
-   [12. UI/UX Direction](#12-uiux-direction)
-   [13. Repository Structure](#13-repository-structure)
-   [14. Local Development](#14-local-development)
-   [15. Testing and Quality](#15-testing-and-quality)
-   [16. Roadmap](#16-roadmap)
-   [17. Architecture Decisions and Reconciliation
    Notes](#17-architecture-decisions-and-reconciliation-notes)
-   [18. Detailed Documentation](#18-detailed-documentation)
-   [19. Contributing](#19-contributing)

------------------------------------------------------------------------

## 1. Vision

### The problem

Anime and manga fans often split their activity across multiple
products:

-   Catalog and tracking services provide structured information,
    ratings, and watchlists, but their discussion experience may feel
    secondary.
-   Social platforms provide active conversations and discovery, but
    titles are often just text and are disconnected from a user's
    progress or personal library.
-   Spoiler-heavy discussions make it difficult to explore a fandom
    safely when users are at different episodes, chapters, or volumes.

### The solution: contextualized social discovery

Anivio connects the content, the conversation, and the fan:

1.  A user discovers a title through a post, community, or
    recommendation.
2.  The title is represented as a structured catalog entity, not merely
    a text string.
3.  Users can open its information card, view related discussions, and
    add it to a personal list.
4.  Users can participate in communities and follow people whose
    interests match theirs.
5.  Spoiler controls help protect readers and viewers at different
    points in a story.

### Product identity

-   **Working product name:** Anivio.
-   **Product category:** Community-driven social discovery platform.
-   **Experience model:** Hybrid of Reddit-style communities and
    social-network profiles/following.
-   **Initial market:** India-first, designed for a global audience and
    international expansion.
-   **Platform direction:** Responsive web application, with installable
    PWA support as an optional enhancement.

The name, domain, and brand identity should be treated as project
identifiers until the team formally confirms the final brand.

------------------------------------------------------------------------

## 2. Product Principles

1.  **Community first:** Discussions and fandom participation are the
    primary experience; catalog data supports the conversation.
2.  **Titles are native entities:** Anime, manga, manhwa, manhua,
    webtoons, light novels, and web novels should be represented as
    structured catalog items.
3.  **Spoiler safety by design:** Spoiler handling is a product-level
    capability, not just a visual effect.
4.  **User-controlled discovery:** The home feed balances followed
    communities and people with relevant discovery.
5.  **Calm visual hierarchy:** Artwork provides visual character; the
    surrounding interface remains readable and restrained.
6.  **Honest interface states:** Loading, empty, error, degraded, and
    permission states must be distinct.
7.  **Modular, not over-engineered:** Begin with a modular monolith and
    extract services only when real scale or operational needs justify
    it.
8.  **Secure defaults:** Authorization, validation, privacy, and
    moderation are enforced on the server.

------------------------------------------------------------------------

## 3. Audience and Launch Strategy

### Audience segments

  -----------------------------------------------------------------------
  Segment                             Needs
  ----------------------------------- -----------------------------------
  Seasonal viewers/readers            Current-season discovery,
                                      episode/chapter conversations,
                                      progress-aware spoiler protection

  Archival fans                       Large personal lists, ratings,
                                      reviews, profile history

  Casual discoverers                  Recommendations, trending
                                      discussions, spoiler-safe browsing

  Community regulars                  Communities, posts, nested
                                      discussions, voting, moderation

  Creators and artists                A place to share fan art and
                                      creative work, subject to content
                                      rules
  -----------------------------------------------------------------------

### Content coverage

The product model is intended to support:

-   Anime
-   Manga
-   Manhwa
-   Manhua
-   Webtoons
-   Light novels
-   Web novels
-   Other related fiction formats where the catalog model supports them

The initial experience can prioritize anime metadata and workflows while
keeping the catalog extensible for the other formats. Do not design the
database around anime-only assumptions where a generic content type is
appropriate.

### Community model

Communities can be organized around:

-   **Series:** a specific anime, manga, manhwa, or franchise.
-   **Genres:** romance, action, fantasy, isekai, psychological, slice
    of life, and more.
-   **General interests:** recommendations, memes, news, fan art,
    reviews, and broader fandom discussions.

Community creation is controlled during the initial launch
(platform-admin provisioned communities). The underlying ownership and
moderator model supports community-level management. A later policy
change may allow eligible trusted users to create communities after
anti-spam and moderation operations are ready.

------------------------------------------------------------------------

## 4. Core Features

### 4.1 Home and discovery

-   Hybrid home feed combining joined communities, followed users, and
    personalized discovery.
-   Following feed for content from followed people and joined
    communities.
-   Discovery surfaces for trending, latest, popular, and relevant
    posts.
-   Filters by content type, community, genre, and post type.
-   Feed controls for muted communities and blocked users.

### 4.2 Communities

-   Community directory and community detail pages.
-   Series-, genre-, and interest-based communities.
-   Membership and leave/join controls.
-   Community description, rules, pinned posts, flairs, and member
    information.
-   Community owner and moderator roles.
-   Reports and moderation actions scoped to the relevant community.

### 4.3 Posts and discussions

Supported post formats include:

-   Discussion
-   Question
-   Theory
-   Review
-   Poll
-   Meme
-   Fan art
-   News
-   Recommendation
-   Text, image, and link posts

Posts support titles, rich text, media attachments, tags/flairs, spoiler
marking, voting, comments, bookmarks, sharing, and reporting.

### 4.4 Comments and voting

-   Nested comments and replies.
-   Upvotes and downvotes.
-   Best, New, and Hot sorting.
-   Optimistic UI updates with server-side validation.
-   Deleted or removed content should preserve discussion continuity
    where appropriate.
-   Comment nesting is bounded to a documented maximum depth.

### 4.5 Structured catalog and title cards

-   Search and browse catalog items.
-   Title detail pages with synopsis, genres, status, artwork, and
    available metadata.
-   Relationships between adaptations, sequels, prequels, and related
    works.
-   Inline title tagging in the post editor (for example, `@Title`).
-   Interactive title cards with a synopsis preview and quick library
    action.
-   Provider attribution and cached metadata snapshots.

### 4.6 Personal library and progress

-   Add titles to a personal library.
-   Generic statuses such as Planned, In Progress, Completed, On Hold,
    and Dropped.
-   UI labels can adapt to the medium (for example, Watching or
    Reading).
-   Track episode, chapter, or volume progress.
-   Ratings, reviews, and personal notes can be introduced in staged
    releases.

### 4.7 Notifications and social graph

-   User profiles and profile activity.
-   Follow/unfollow users.
-   In-app notifications for relevant replies, mentions, and moderation
    events.
-   Direct messages and real-time chat are later-stage capabilities, not
    MVP dependencies.

### 4.8 Spoiler controls

-   Manual spoiler flags on posts and comments.
-   Explicit spoiler reveal interaction.
-   Future progress-aware spoiler checks using the user's tracked
    episode/chapter/volume.
-   Reporting for incorrectly tagged spoilers.

------------------------------------------------------------------------

## 5. MVP Scope

The MVP should validate the core loop: **discover a community → join →
read or create a discussion → reply and vote → return to a useful
feed**.

### P0 --- Launch foundation

-   Responsive web application.
-   OAuth sign-in with Google and Discord.
-   Username onboarding and public profiles.
-   Community directory and admin-provisioned communities.
-   Join/leave membership.
-   Text-first posts and comments/replies.
-   Voting and basic sorting.
-   Home and community feeds.
-   Manual spoiler marking and safe reveal behavior.
-   Basic reporting and moderation queue.
-   Basic title discovery and title detail pages, using cached metadata
    where available.
-   Server-side authorization and rate limits.

### P1 --- Early improvements

-   Image uploads through object storage.
-   Inline title tagging and interactive title cards.
-   Personal watch/read list and progress tracking.
-   Search improvements and trending feed.
-   Bookmarks and user following.
-   In-app notifications.
-   Progress-aware spoiler handling.

### P2 --- Expansion

-   Ratings and long-form reviews.
-   Richer recommendations and personalization.
-   Polls, creator/fan-art features, and community events.
-   Additional catalog formats and provider integrations.
-   More advanced moderation and community analytics.

### P3 --- Optional future capabilities

-   Direct messages and real-time chat rooms.
-   Advanced recommendation models.
-   Mobile-native applications, if validated by usage.
-   Further social and creator tools.

**Scope rule:** Do not make every planned feature a launch blocker. Ship
a safe, stable discussion loop before adding high-complexity social
features.

------------------------------------------------------------------------

## 6. Technology Stack

The current technical baseline is a TypeScript-first Next.js modular
monolith.

  -----------------------------------------------------------------------
  Layer                   Selected technology     Responsibility
  ----------------------- ----------------------- -----------------------
  Web framework           Next.js 16 App Router   Routing, server
                                                  rendering, server
                                                  actions, route handlers

  Language                TypeScript 5.x          Strict typing across
                                                  application code

  UI                      React Server Components Server-first rendering
                          by default              and reduced client
                                                  JavaScript

  Styling                 Tailwind CSS 3.4.x      Utility styling and
                                                  design-token
                                                  integration

  UI primitives           shadcn/ui               Accessible, composable
                                                  UI primitives

  Forms/validation        Zod 3.23.x              Runtime validation and
                                                  typed input contracts

  Rich-text editor        Tiptap 2.x              Restricted-schema post
                                                  composition

  Database                PostgreSQL 16+ on Neon  Relational persistence
                                                  and full-text search

  ORM/query layer         Drizzle ORM 0.36.x      Typed SQL access and
                                                  migrations

  Authentication          Auth.js v5 baseline     Google/Discord OAuth
                                                  and database-backed
                                                  sessions

  Cache/rate limits       Upstash Redis           Distributed rate
                                                  limiting and selected
                                                  caching

  Media storage           Cloudflare R2           Presigned direct
                                                  uploads and media
                                                  delivery

  Anime metadata          Tenrai API v1 primary;  External catalog
                          AniList GraphQL         metadata
                          fallback                

  Testing                 Vitest, Playwright      Unit, integration, and
                                                  browser tests

  Monitoring              Sentry                  Error tracking and
                                                  performance telemetry

  Hosting                 Vercel                  Web deployment and edge
                                                  delivery
  -----------------------------------------------------------------------

Versions above reproduce the supplied architecture baseline. Before
implementation or upgrades, verify package compatibility and provider
availability in the provider-spike milestone. Avoid changing a locked
technology without recording the reason in an architecture decision
record (ADR).

------------------------------------------------------------------------

## 7. System Architecture

### Architecture style

Anivio uses a **domain-driven modular monolith**. The application is
deployed as one Next.js application, while business logic is separated
into explicit domain modules. This keeps deployment and development
manageable while protecting boundaries between features.

``` mermaid
flowchart TD
    User["Browser / Mobile Web"] --> Edge["Vercel Edge / CDN"]
    Edge --> Next["Next.js 16 App Router"]
    Next --> UI["Server Components and Client Islands"]
    Next --> Actions["Server Actions"]
    Next --> Routes["Route Handlers"]
    Actions --> Domains["Domain Modules"]
    Routes --> Domains
    Domains --> Drizzle["Drizzle ORM"]
    Drizzle --> DB[("Neon PostgreSQL 16+")]
    Domains --> Redis[("Upstash Redis")]
    Domains --> R2[("Cloudflare R2")]
    Metadata["Metadata Adapter"] --> Tenrai["Tenrai API"]
    Metadata --> AniList["AniList GraphQL"]
    Domains --> Metadata
    Auth["Auth.js"] --> OAuth["Google / Discord OAuth"]
    Domains --> Auth
    Next --> Sentry["Sentry"]
```

### Domain modules

-   `auth` --- sessions, OAuth integration, onboarding, access guards.
-   `users` --- profiles, settings, follows, blocks.
-   `communities` --- community data, membership, roles, rules.
-   `discussions` --- posts, comments, votes, ranking.
-   `catalog` / `metadata` --- normalized catalog entities, provider
    adapters, snapshots.
-   `watchlist` / `library` --- user lists and progress.
-   `moderation` --- reports, enforcement, audit history.
-   `notifications` --- in-app notifications.
-   `jobs` --- scheduled and asynchronous work.

### Server-first execution

-   React Server Components are the default.
-   Client Components are reserved for interactions requiring browser
    state (editor, voting controls, menus, dialogs, progress controls).
-   Database access and secrets remain server-side.
-   Server Actions handle authenticated mutations; Route Handlers expose
    endpoints needed by search, uploads, integrations, and external
    clients.
-   Every mutation validates input and authorization on the server.

### Boundary rules

1.  `src/lib/` contains infrastructure clients and must not import
    domain modules or UI.
2.  `src/components/ui/` contains reusable primitives and must not
    access domain data.
3.  `src/modules/` contains business logic and data access, never
    presentation components.
4.  `src/app/` composes pages and invokes domain operations; domain
    modules must not import from `src/app/`.
5.  Cross-module dependencies should use narrow exported functions or
    typed contracts, not deep imports into another module's internals.

------------------------------------------------------------------------

## 8. Core Domain Model

The detailed SQL schema and constraints live in `BACKEND.md`. At a
conceptual level, the system includes:

  -----------------------------------------------------------------------
  Domain                              Main entities
  ----------------------------------- -----------------------------------
  Identity                            Users, OAuth accounts, sessions,
                                      profiles, settings

  Social graph                        User follows, blocks

  Communities                         Communities, memberships, rules,
                                      flairs

  Discussions                         Posts, media, tags, comments, post
                                      votes, comment votes

  Catalog                             Franchises, catalog items, genres,
                                      relations,
                                      episodes/chapters/volumes

  Personal tracking                   User library/watchlists, progress,
                                      reviews

  Safety                              Reports, moderation actions, audit
                                      logs

  Engagement                          Bookmarks, notifications,
                                      karma/counters
  -----------------------------------------------------------------------

### Data principles

-   Use UUID primary keys for application entities unless an external
    identifier is explicitly required.
-   Store timestamps in UTC (`timestamptz`).
-   Enforce relationships and uniqueness with database constraints.
-   Use transactions for multi-step mutations such as voting,
    membership, and moderation.
-   Keep post votes and comment votes separately when conventional
    foreign-key integrity is important.
-   Treat denormalized counters and ranking values as derived data that
    can be recalculated.
-   Use cursor/keyset pagination for feeds and large comment
    collections.
-   Use PostgreSQL full-text search and trigram matching as the initial
    search layer.
-   Keep external provider IDs in dedicated mapping fields/tables; do
    not make one provider's ID the universal identity for every catalog
    record.

### Comment hierarchy

The supplied backend baseline uses PostgreSQL `ltree` paths for
hierarchical comments, with transactional insertion and a maximum
nesting depth. This is retained as the selected design. The
implementation must ensure path construction, parent validation, depth
enforcement, and vote/comment count updates are transactional.

------------------------------------------------------------------------

## 9. Feed and Ranking

The home feed is a hybrid of:

1.  **Following:** posts from joined communities and followed users.
2.  **Discovery:** posts relevant to selected interests, genres, and
    catalog activity.
3.  **Trending:** posts with strong recent engagement.

Initial ranking should be deterministic and explainable rather than
dependent on a machine-learning model. Candidate signals include:

-   Net votes and vote quality.
-   Comment/reply activity.
-   Recency and time decay.
-   Relationship to followed communities and users.
-   Interest or content-category relevance.
-   Content eligibility, moderation state, blocks, and mutes.

Use normalized signals or a clearly defined ranking function so one
high-scale raw metric does not overwhelm all others. Add diversity
controls to avoid showing near-identical posts repeatedly. Provide
chronological sorting as a user-visible alternative.

The supplied technical baseline includes Wilson Score ranking for
quality-oriented sorting and a Hot Rank time-decay function. Their exact
formulas and SQL implementations belong in `BACKEND.md`.

------------------------------------------------------------------------

## 10. Spoiler Protection

Spoiler protection is a core trust feature.

### Required behavior

-   Posts and comments can be explicitly marked as spoilers.
-   Hidden content is concealed until the user chooses to reveal it.
-   Spoiler labels and reveal controls must be accessible by keyboard
    and screen reader.
-   Users can report content that is incorrectly tagged.
-   If progress-aware spoiler protection is enabled, compare the
    referenced story unit with the user's tracked progress.

### Security requirement

**Do not rely on CSS blur or frontend hiding alone.** When content must
be withheld, the server must apply the spoiler policy before returning
protected content. The API/Server Action response must not expose hidden
text or media metadata that defeats the protection. Manual reveal should
request or unlock the content through an authorized server path.

Progress-aware protection should be introduced only when catalog unit
mappings are reliable. Unknown or ambiguous progress should fail safely
and preserve manual spoiler controls.

------------------------------------------------------------------------

## 11. Security and Moderation

### Authentication

-   Google and Discord OAuth through Auth.js with PKCE.
-   PostgreSQL-backed sessions.
-   Cookies must be `HttpOnly`, `Secure` in production, and
    appropriately `SameSite`.
-   Session revocation must take effect when the corresponding database
    session is removed.
-   Username selection and uniqueness are enforced server-side.

### Authorization

Use distinct global and community roles:

-   Global roles: `user`, `moderator`, `admin` (and super-admin if
    required by operations).
-   Community roles: `member`, `moderator`, `owner`.

Role checks must be performed server-side for every protected operation.
UI visibility is not authorization.

### Content and application defenses

-   Parameterized database queries; no unsafe SQL string concatenation.
-   Validate all input using shared Zod schemas.
-   Restrict rich-text nodes and marks; sanitize rendered content.
-   Reject unsupported upload MIME types, excessive file sizes, and
    invalid dimensions.
-   Use presigned uploads so large media does not pass through
    application compute.
-   Apply rate limits to account creation, posting, commenting, voting,
    search, and uploads.
-   Protect new accounts with stricter limits and anti-spam controls.
-   Keep secrets in environment variables; never commit `.env` files.
-   Maintain moderation and administrative audit records.
-   Define privacy, account deletion, content reporting, appeals, and
    copyright/takedown procedures before broad public launch.

### Moderation workflow

-   Users can report posts, comments, and accounts.
-   Reports enter a review queue.
-   Moderators can remove or filter content within their community
    permissions.
-   Global administrators can suspend or ban accounts and take
    platform-wide actions.
-   Automated report thresholds may temporarily filter content, but
    should not replace human review or provide an irreversible
    punishment by themselves.
-   Every enforcement action records actor, target, reason, timestamp,
    and outcome.

The detailed RBAC matrix, rate limits, audit requirements, and
deployment controls are documented in `OPERATIONS.md`.

------------------------------------------------------------------------

## 12. UI/UX Direction

### Visual direction

Anivio uses a calm, content-first dark interface. Anime artwork is the
primary source of visual color; interface accents communicate
interaction and state rather than decoration.

### Design rules

-   Mobile-first responsive layout.
-   Reading column target width: approximately 720px.
-   Four-pixel spacing rhythm.
-   Consistent surface, border, typography, and state tokens.
-   Avoid raw color values inside components; use CSS design tokens.
-   Keyboard-visible focus states and accessible controls.
-   Touch targets should be comfortably sized (the supplied design
    baseline targets at least 44 × 44px).
-   Respect `prefers-reduced-motion`.
-   Provide distinct loading, empty, error, degraded, and permission
    states.

The complete palette, typography, spacing, component anatomy, responsive
page layouts, and accessibility checklist are defined in `DESIGN.md`.

------------------------------------------------------------------------

## 13. Repository Structure

The target repository layout follows the supplied architecture baseline:

``` text
anivio/
├── README.md
├── docs/
│   ├── README.md
│   ├── PRODUCT_SPEC.md
│   ├── ARCHITECTURE.md
│   ├── BACKEND.md
│   ├── DESIGN.md
│   ├── OPERATIONS.md
│   └── DEVELOPMENT.md
├── drizzle/                  # Forward-only SQL migrations
├── scripts/                  # Idempotent seed and maintenance scripts
├── public/                   # Static assets
└── src/
    ├── app/                  # Next.js App Router
    │   ├── (marketing)/
    │   ├── (main)/
    │   ├── (auth)/
    │   ├── mod/
    │   └── api/
    ├── components/
    │   ├── ui/
    │   ├── anime/
    │   ├── discussion/
    │   ├── community/
    │   └── feedback/
    ├── modules/
    │   ├── auth/
    │   ├── users/
    │   ├── metadata/
    │   ├── communities/
    │   ├── discussions/
    │   ├── watchlist/
    │   ├── moderation/
    │   └── jobs/
    ├── lib/                  # DB, Redis, storage, env clients
    └── styles/               # Design tokens and global styles
```

The repository may evolve as implementation reveals better boundaries,
but changes must be reflected in `ARCHITECTURE.md`.

------------------------------------------------------------------------

## 14. Local Development

### Prerequisites

-   Node.js version supported by the selected Next.js release.
-   npm, pnpm, or the package manager committed by the project.
-   PostgreSQL/Neon database credentials.
-   OAuth application credentials for Google and Discord.
-   Optional service credentials for Upstash Redis and Cloudflare R2.

### Setup

The exact scripts depend on the committed `package.json`. The general
setup sequence is:

1.  Clone the repository.
2.  Install dependencies using the repository's declared package
    manager.
3.  Copy `.env.example` to `.env.local`.
4.  Fill in the required environment variables.
5.  Apply the numbered Drizzle SQL migrations.
6.  Run the development server using the package script.
7.  Run the test and lint scripts before opening a pull request.

Do not invent or assume script names: keep this section synchronized
with the actual `package.json` once the implementation repository is
available.

### Environment configuration

The supplied operations baseline defines environment variables for:

-   `DATABASE_URL`
-   `AUTH_SECRET`
-   `AUTH_GOOGLE_ID`
-   `AUTH_GOOGLE_SECRET`
-   `AUTH_DISCORD_ID`
-   `AUTH_DISCORD_SECRET`
-   `UPSTASH_REDIS_REST_URL`
-   `UPSTASH_REDIS_REST_TOKEN`
-   Cloudflare R2 account, bucket, access-key, and secret values
-   Sentry configuration where enabled
-   Metadata provider configuration where required

Use `.env.example` for names and safe placeholders only. Never place
real credentials in documentation or source control.

------------------------------------------------------------------------

## 15. Testing and Quality

### Test layers

  -----------------------------------------------------------------------
  Layer                   Tool                    Main coverage
  ----------------------- ----------------------- -----------------------
  Unit                    Vitest                  Ranking functions,
                                                  validators,
                                                  normalizers, utility
                                                  logic

  Integration             Vitest + PostgreSQL     Transactions, comment
                                                  hierarchy, votes,
                                                  sessions, snapshots

  End-to-end              Playwright              Sign-in/onboarding,
                                                  community
                                                  participation,
                                                  comments/voting,
                                                  tracking

  Failure testing         Scripts                 Metadata provider
                                                  outage, Redis outage,
                                                  database degraded mode

  Accessibility           Axe and manual checks   Keyboard navigation,
                                                  focus, contrast,
                                                  screen-reader behavior

  Load                    k6                      Feed reads and large
                                                  comment threads
  -----------------------------------------------------------------------

### Critical user journeys

1.  Sign in and complete username onboarding.
2.  Discover and join a community.
3.  Create a post and publish it.
4.  Reply to a discussion and vote.
5.  Open a title card and add the title to the personal list.
6.  Report content and verify moderation permissions.

### Quality gates

-   Protected actions reject unauthorized users.
-   Duplicate votes and memberships are prevented.
-   Ranking and counters remain consistent after mutations.
-   Spoiler-protected content is not leaked by server responses.
-   Pagination is stable and does not require unbounded offset scans.
-   Core pages work on mobile and desktop.
-   Accessibility and performance budgets are checked before release.
-   Database migrations are reviewed and tested before production
    deployment.

------------------------------------------------------------------------

## 16. Roadmap

The supplied development plan defines milestones M0--M9 and estimates
approximately 310 focused hours across 17 weeks for a solo developer.
Treat these as planning estimates, not delivery promises.

  -----------------------------------------------------------------------
  Milestone               Focus                   Outcome
  ----------------------- ----------------------- -----------------------
  M0                      Provider spike          Validate metadata
                                                  providers, save
                                                  fixtures, define
                                                  normalized catalog
                                                  contract

  M1                      Foundation              Next.js app, design
                                                  tokens, database
                                                  extensions, CI, initial
                                                  deployment

  M2                      Identity                OAuth, database
                                                  sessions, onboarding,
                                                  profiles, account
                                                  lifecycle

  M3                      Data layer              Metadata adapters,
                                                  snapshots,
                                                  catalog/search pages

  M4                      Social core             Communities,
                                                  membership, post
                                                  creation, media upload
                                                  foundation

  M5                      Discussion engine       Nested comments, votes,
                                                  sorting, ranking

  M6                      Tracking and tagging    Personal lists,
                                                  progress, inline title
                                                  tags

  M7                      Trust and safety        Reports, moderation
                                                  queue, role
                                                  enforcement, audit log

  M8                      Search and hardening    Full-text search,
                                                  notifications,
                                                  accessibility and
                                                  performance audit

  M9                      Launch                  Production setup, seed
                                                  communities, beta
                                                  onboarding, launch
                                                  monitoring
  -----------------------------------------------------------------------

### Six-week contingency

If time is constrained, reduce scope rather than removing security
fundamentals:

-   Weeks 1--2: foundation and identity.
-   Weeks 3--4: metadata basics and minimal community/post
    functionality.
-   Week 5: simplified comments and voting.
-   Week 6: basic list tracking, manual QA, and deployment.

Defer advanced ranking, entity tagging, rich media, notifications, and
advanced moderation automation rather than shipping unsafe or untested
behavior.

------------------------------------------------------------------------

## 17. Architecture Decisions and Reconciliation Notes

This README consolidates the uploaded project documentation with the
earlier product-planning direction. The detailed source documents remain
authoritative for implementation-level specifications. The following
choices make the combined direction explicit.

### Decisions retained from the uploaded baseline

-   Next.js 16 App Router as the full-stack application framework.
-   Domain-driven modular monolith rather than microservices.
-   PostgreSQL 16+ on Neon and Drizzle ORM.
-   Auth.js with Google/Discord OAuth and database sessions.
-   PostgreSQL `ltree` for comment hierarchy.
-   Tiptap with a restricted rich-text schema.
-   Upstash Redis for distributed rate limits and selected caching.
-   Cloudflare R2 for media uploads.
-   Tenrai primary metadata adapter and AniList fallback, subject to
    provider validation.
-   PostgreSQL full-text and trigram search as the initial search
    system.
-   Vitest and Playwright for automated testing.
-   Vercel deployment and Sentry monitoring.

### Product decisions clarified in this merged version

  -----------------------------------------------------------------------
  Topic                               Consolidated direction
  ----------------------------------- -----------------------------------
  Product experience                  Reddit-style communities plus
                                      social profiles and following

  Home feed                           Hybrid: followed/joined content
                                      plus personalized discovery

  Content scope                       Extensible across anime, manga,
                                      manhwa, manhua, webtoons, light
                                      novels, and web novels; anime-first
                                      metadata implementation

  Community creation                  Admin-provisioned for initial
                                      launch; architecture retains
                                      owner/moderator roles and can
                                      support trusted-user creation later

  Launch platform                     Responsive web first; PWA is
                                      optional; native mobile apps are
                                      not an MVP dependency

  Spoilers                            Manual spoiler controls in the core
                                      experience; progress-aware
                                      filtering introduced when catalog
                                      unit data is reliable

  Delivery strategy                   Modular monolith, staged MVP, no
                                      premature microservices or
                                      Kubernetes
  -----------------------------------------------------------------------

### Important inconsistencies to keep resolved

1.  **Community creation:** The supplied `PRODUCT_SPEC.md` and
    `OPERATIONS.md` currently describe eligible users creating
    communities after an account-age threshold, while the earlier
    product direction selected admin-only creation at launch. This
    README uses admin-only creation for the initial release. Update
    `FR-010`, the RBAC matrix, and milestone M4 together before
    implementation so the detailed documents do not contradict this
    launch policy.
2.  **Content coverage:** The supplied documents are anime-centric and
    use MAL-compatible identifiers. The broader product direction
    includes multiple fiction formats. Keep anime as the first metadata
    integration, but use generic catalog item types and unit types
    before extending to other media.
3.  **Architecture stack:** Earlier planning considered NestJS/Prisma
    and a separate API. The uploaded architecture baseline is more
    specific and consistently defines Next.js/Drizzle/Auth.js. This
    merged README retains the uploaded baseline; do not mix both stacks
    in one implementation.
4.  **Spoiler behavior:** Frontend-only blur is insufficient.
    Server-side response filtering is required wherever a user's spoiler
    policy means the content must remain hidden.
5.  **Metadata providers:** Provider availability, quotas, attribution
    rules, and terms must be checked during M0. Do not rely on scraping
    or bypass provider restrictions.
6.  **MVP boundaries:** The source specifications contain several P0/P1
    assignments. The MVP section above prioritizes a safe community
    discussion loop; title tagging, media uploads, advanced tracking,
    and advanced personalization can be staged without blocking the
    first usable release.

Any change to a locked technical choice should be recorded as an ADR and
reflected in the relevant detailed document.

------------------------------------------------------------------------

## 18. Detailed Documentation

The repository's `docs/` directory contains the full specifications.
This README is the orientation and project overview; it does not replace
the detailed contracts.

  ------------------------------------------------------------------------------------
  Document                                         Authority
  ------------------------------------------------ -----------------------------------
  [`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md)   Product vision, functional
                                                   requirements, non-functional
                                                   requirements, workflows

  [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)   Technology choices, repository
                                                   structure, module boundaries, ADRs

  [`docs/BACKEND.md`](docs/BACKEND.md)             Database DDL, ERD, stored
                                                   functions, server actions, route
                                                   handlers, metadata adapters

  [`docs/DESIGN.md`](docs/DESIGN.md)               Design tokens, typography,
                                                   components, page layouts,
                                                   accessibility

  [`docs/OPERATIONS.md`](docs/OPERATIONS.md)       Authentication security, RBAC,
                                                   abuse prevention, tests,
                                                   infrastructure, CI/CD, recovery

  [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md)     Milestones, task board, engineering
                                                   standards, Git workflow, changelog
  ------------------------------------------------------------------------------------

### Documentation ownership rules

1.  Each technical requirement has one authoritative home.
2.  Feature, schema, API, security, and design changes must update the
    corresponding document in the same pull request.
3.  Tasks should trace to stable identifiers such as `FR-xxx`,
    `NFR-xxx`, `TASK-xxx`, and `ADR-xxx`.
4.  If two documents conflict, resolve the conflict explicitly and
    update both; do not silently choose whichever is convenient.

------------------------------------------------------------------------

## 19. Contributing

1.  Create a focused branch using the project naming convention
    (`feat/`, `fix/`, `chore/`, `docs/`, or `perf/`).
2.  Use Conventional Commits.
3.  Keep domain boundaries intact.
4.  Validate inputs and permissions on the server.
5.  Add or update tests for behavior changes.
6.  Keep migrations forward-only and review them before production.
7.  Update the relevant documentation with the code change.
8.  Never commit secrets, personal data, or production database dumps.

### Pull request checklist

-   [ ] Requirements and acceptance criteria are clear.
-   [ ] Domain boundaries and architecture decisions are respected.
-   [ ] Inputs are validated and protected actions are authorized
    server-side.
-   [ ] Database changes are safe, reviewed, and tested.
-   [ ] Tests cover the changed behavior.
-   [ ] Mobile layout, keyboard access, and focus states are checked.
-   [ ] Performance impact is understood.
-   [ ] Relevant documentation is updated.

------------------------------------------------------------------------

## License and legal notes

No license is specified in the supplied project documents. Add an
explicit license before accepting external contributions. Anivio should
store metadata and user-created content in accordance with provider
terms and applicable law; it should not host or redistribute copyrighted
anime, manga, or novel chapters/episodes without authorization.
