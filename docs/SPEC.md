# Anivio — Functional & Non-Functional Specifications

> **Single Source of Truth (SSOT) for Feature Specifications**  
> Status: **LOCKED BASELINE (Version 1.0)**  
> Last Updated: 12 November 2026

This document defines every functional feature and non-functional requirement for the Anivio Anime Community Platform. All development, UI design, database implementation, and testing must trace directly to the requirement IDs defined here.

- For UI token values and component layouts, see [DESIGN.md](DESIGN.md).
- For step-by-step user interaction flows and sequence diagrams, see [WORKFLOW.md](WORKFLOW.md).
- For database schema and constraints, see [DATABASE.md](DATABASE.md).
- For server action and route handler signatures, see [API.md](API.md).
- For architectural decisions and open items, see [DECISIONS.md](DECISIONS.md).

---

## Specification Conventions

- **Requirement IDs**: `FR-xxx` (Functional Requirement), `NFR-xxx` (Non-Functional Requirement).
- **Priorities**:
  - **P0 (Essential MVP)**: Mandatory for initial launch. Launch is blocked if incomplete.
  - **P1 (Important)**: Targeted for Version 1 launch; required for full core experience.
  - **P2 (Post-Launch)**: Scheduled for post-MVP iteration.
- **Traceability**: Every requirement includes Role, Preconditions, Expected Behaviour, Validation Rules, Success/Error States, Edge Cases, Dependencies, and Testable Acceptance Criteria.

---

## 1. Authentication & Onboarding Module

### FR-001: OAuth 2.0 Authentication
- **Priority**: P0
- **User Roles**: Anonymous Visitor, Registered User
- **Preconditions**: User has a valid Google or Discord account.
- **Expected Behaviour**:
  1. Visitor clicks "Sign in with Google" or "Sign in with Discord".
  2. System initiates OAuth 2.0 flow with PKCE via Auth.js.
  3. Callback validates provider tokens, provisions or retrieves database session.
  4. Session cookie is issued (`__Host-` or `authjs.session-token`, `HttpOnly`, `Secure`, `SameSite=Lax`).
  5. If user is new, system redirects to `/auth/onboarding` (FR-002). If existing, system redirects to preserved return URL.
- **Validation Rules**:
  - No password-based authentication permitted.
  - OAuth redirect URIs must be exact matches per environment (no wildcards).
- **Success State**: Valid session row inserted in `sessions` table; client receives authenticated cookie; user is redirected to destination.
- **Error States**:
  - Provider denies authorization: redirect to `/auth/signin` with error alert "Sign-in cancelled".
  - OAuth token exchange failure: return HTTP 401 with `Unauthenticated` code.
- **Dependencies**: Auth.js, PostgreSQL `accounts` and `sessions` tables, Google Cloud Console, Discord Developer Portal.
- **Acceptance Criteria**:
  - [ ] Logging in via Google provisions a persistent database session.
  - [ ] Logging in via Discord provisions a persistent database session.
  - [ ] Session cookie cannot be read via JavaScript `document.cookie`.
  - [ ] Revoking session in database logs out client on next request.

### FR-002: Username Selection & Onboarding
- **Priority**: P0
- **User Roles**: Newly Authenticated User
- **Preconditions**: User authenticated via OAuth but lacks a confirmed `username`.
- **Expected Behaviour**:
  1. User is prompted to select a unique username and optional display name.
  2. Live debounced availability check indicates availability.
  3. Form submission persists username in `users` table with role `user`.
- **Validation Rules**:
  - Username regex: `^[a-zA-Z0-9_-]{3,24}$` (3–24 characters, alphanumeric, hyphen, underscore).
  - Case-insensitive uniqueness enforced via PostgreSQL `citext`.
  - Reserved names blocked: `admin`, `mod`, `moderator`, `anivio`, `api`, `system`, `root`, `help`, `support`, `settings`.
- **Success State**: User row updated with unique username; onboarding marked complete; redirect to return path.
- **Error States**:
  - Username already taken: return `Conflict` ("Username is already in use").
  - Invalid format: return `ValidationFailed` with specific constraint message.
- **Dependencies**: PostgreSQL `users` table with `citext`.
- **Acceptance Criteria**:
  - [ ] Case-insensitive collisions (e.g., `Spike` vs `spike`) are rejected at database level.
  - [ ] System blocks navigation to other platform features until onboarding completes.

### FR-003: Account Settings & Self-Deletion
- **Priority**: P0
- **User Roles**: Registered User (Self)
- **Preconditions**: Active user session.
- **Expected Behaviour**:
  1. User can update avatar URL and display preferences.
  2. User can trigger account deletion in Danger Zone by typing exact confirmation text `DELETE MY ACCOUNT`.
  3. Single database transaction deletes sessions, accounts, and anonymizes user row (`username` replaced with `[deleted_user_<id>]`, email nullified, `is_deleted` set). Authored posts and comments remain attributed to anonymized identifier.
- **Success State**: All active sessions destroyed, user row anonymized, redirect to home page with goodbye toast.
- **Error States**: Confirmation string mismatch returns `ValidationFailed`.
- **Dependencies**: PostgreSQL cascade rules and anonymization transaction.
- **Acceptance Criteria**:
  - [ ] Account deletion completely removes OAuth linkage and active sessions.
  - [ ] Post and comment trees remain visually intact with author rendered as `[deleted]`.

---

## 2. User Profiles Module

### FR-004: Public User Profile
- **Priority**: P0
- **User Roles**: All Users, Anonymous Visitors
- **Preconditions**: Profile exists at `/u/[username]`.
- **Expected Behaviour**: Displays user avatar, username, total karma, account age, and chronological tabbed feed of public posts and comments.
- **Caching**: Framework-cached for 60 seconds with on-demand tag revalidation upon profile update.
- **Dependencies**: `users`, `posts`, `comments` tables.
- **Acceptance Criteria**:
  - [ ] Visiting `/u/[username]` renders avatar, karma, and public posts within 2.5s.
  - [ ] Non-existent username renders standard 404 page.

### FR-005: Karma Computation & Display
- **Priority**: P0
- **User Roles**: Registered User
- **Preconditions**: User has authored posts or comments that receive upvotes or downvotes.
- **Expected Behaviour**: Karma reflects the sum of net votes received on authored posts and comments.
- **Implementation**: Stored as a denormalized integer column on `users` table, updated via background job coalescing (`recomputeScore`), with a daily scheduled repair job (`repairCounters`).
- **Acceptance Criteria**:
  - [ ] Upvoting a post increases author karma by 1.
  - [ ] Retracting upvote decrements author karma by 1.
  - [ ] Daily counter repair script reconciles drifted karma values.

---

## 3. Anime Discovery Module

### FR-006: Current Season Grid
- **Priority**: P0
- **User Roles**: All Users, Anonymous Visitors
- **Preconditions**: Route `/season` accessed.
- **Expected Behaviour**:
  1. Displays responsive poster grid (2 columns on mobile, 3 on tablet, 4–6 on desktop) of currently airing anime.
  2. Each card renders poster (2:3 aspect ratio), title (max 2 lines with ellipsis), genre pills, and score chip.
  3. Wrapped in React Suspense with layout-matched skeleton loaders.
- **Caching**: 24 hours via Next.js `'use cache'`, refreshed by daily background cron job.
- **Resilience**: If provider fails, data is served from `provider_snapshots` with a `DegradedNotice` displaying data age.
- **Acceptance Criteria**:
  - [ ] `/season` renders within performance budget (<2.5s LCP on 4G).
  - [ ] Layout shift (CLS) is <0.1 during image/data load.

### FR-007: Top Anime Ranking
- **Priority**: P0
- **User Roles**: All Users, Anonymous Visitors
- **Preconditions**: Route `/top` accessed.
- **Expected Behaviour**: Paginated ranked list of all-time highest-rated anime with pagination controls.
- **Caching**: 24 hours via Next.js `'use cache'`.
- **Acceptance Criteria**:
  - [ ] Displays numerical ranking rank #1 through #N.
  - [ ] Page changes update URL search params (`?page=2`) with keyset or page pagination.

### FR-008: Anime Search Combobox & Dedicated Search
- **Priority**: P0
- **User Roles**: All Users, Anonymous Visitors
- **Preconditions**: Input provided to search bar at `/search?q=` or inline editor tag combobox.
- **Expected Behaviour**:
  1. Debounced (300ms) request to `/api/search/anime?q=query`.
  2. Returns keyboard-navigable listbox with poster thumbnail, title, year, and score.
  3. Selecting an item navigates to `/anime/[malId]`.
- **Rate Limit & Caching**:
  - 30 requests/min per IP.
  - Search results cached for 1 hour; negative results (empty) cached for 5 minutes.
- **Acceptance Criteria**:
  - [ ] Typing query fires no more than 1 request per 300ms.
  - [ ] Keyboard navigation (ArrowDown, ArrowUp, Enter, Escape) functions per WCAG 2.1 AA.

### FR-009: Show Detail View
- **Priority**: P0
- **User Roles**: All Users, Anonymous Visitors
- **Preconditions**: Route `/anime/[malId]` accessed.
- **Expected Behaviour**:
  1. Renders banner/poster, official English and Romanized titles, synopsis excerpt, genres, studio, episode count, broadcast status, and community rating.
  2. Prominently features the MyAnimeList data attribution notice.
  3. Embeds `WatchlistControl` component (FR-030).
  4. Renders list of episodes linking to episode-specific discussion threads.
  5. Renders related community posts tagged with this anime entity.
- **Caching**: 24 hours, backed by `provider_snapshots`.
- **Acceptance Criteria**:
  - [ ] Show page renders from snapshot with `DegradedNotice` when provider endpoint is unreachable.
  - [ ] MAL attribution is present and visible.

---

## 4. Communities Module

### FR-010: Community Creation
- **Priority**: P0
- **User Roles**: Registered User (Account age >= 3 days)
- **Preconditions**: User signed in, not rate-limited.
- **Expected Behaviour**:
  1. User submits unique slug, display name, and description.
  2. Transaction inserts row into `communities` and assigns creating user role `owner` in `community_members`.
  3. Redirects to `/c/[slug]`.
- **Validation Rules**:
  - Slug: 3–24 characters, `^[a-z0-9-]+$`.
  - Name: 3–50 characters.
  - Description: 10–500 characters.
- **Acceptance Criteria**:
  - [ ] Account younger than 3 days receives `Forbidden` ("Account must be 3 days old to create a community").
  - [ ] Duplicate slug returns `Conflict` ("Community slug already exists").

### FR-011: Community Feed & Navigation
- **Priority**: P0
- **User Roles**: All Users
- **Preconditions**: Route `/c/[slug]` accessed.
- **Expected Behaviour**:
  1. Displays community header: name, description, member count, and join/leave toggle button.
  2. Tabbed sorting: `Hot` (default), `New`, `Best`.
  3. Keyset-paginated post stream.
  4. Sidebar displays community rules, moderator list, and "Create Post" button.
- **Acceptance Criteria**:
  - [ ] Feed pagination uses cursor tokens without duplicate or skipped posts.
  - [ ] Unauthenticated visitor can view feed and sidebar.

### FR-012: Community Membership Management
- **Priority**: P0
- **User Roles**: Registered User
- **Preconditions**: User signed in.
- **Expected Behaviour**:
  1. User clicks "Join" or "Leave" on `/c/[slug]`.
  2. Optimistic UI update toggles button label and increments/decrements member counter.
  3. Server action executes idempotent insert/delete in `community_members`.
  4. Rolled back with toast message if server action fails.
- **Acceptance Criteria**:
  - [ ] Rapid multiple clicks result in exactly one consistent database state.
  - [ ] Joined communities populate user's personalized home feed.

---

## 5. Posts & Discussions Module

### FR-013: Rich-Text Post Creation
- **Priority**: P0
- **User Roles**: Community Member
- **Preconditions**: User is signed in, member of community, account >24h old for link inclusion, not shadow-filtered.
- **Expected Behaviour**:
  1. User provides Title (3–300 chars) and body via Tiptap editor.
  2. User can attach up to 4 images (FR-015) and toggle `is_spoiler`.
  3. User can tag an anime entity using `@` trigger (FR-018).
  4. Server action validates payload, parses structured JSON, extracts plaintext `content_text` for search indexing, persists post and tag rows in a single transaction.
  5. Enqueues search vector update and revalidates `/c/[slug]`.
- **Validation Rules**:
  - Title: 3–300 characters.
  - Content JSON: max 50KB structured document.
  - Media: maximum 4 attached IDs.
- **Acceptance Criteria**:
  - [ ] Submitting valid post redirects to `/c/[slug]/p/[postId]`.
  - [ ] Non-member submission fails with `Forbidden`.

### FR-014: Post Editing Window
- **Priority**: P0
- **User Roles**: Post Author, Community Moderator
- **Preconditions**: Post exists, user is author within 24 hours of creation OR user is moderator.
- **Expected Behaviour**:
  1. Author can edit title, content body, and tags.
  2. System updates `edited_at` timestamp and displays `(edited)` indicator on post card.
- **Acceptance Criteria**:
  - [ ] Author cannot edit post after 24 hours elapsed.
  - [ ] Moderator can edit or lock post at any time.

### FR-015: Post Media Direct Upload (Cloudflare R2)
- **Priority**: P1
- **User Roles**: Registered User
- **Preconditions**: User creating or editing post.
- **Expected Behaviour**:
  1. Client requests presigned upload URL from `/api/upload/sign`.
  2. Server verifies authentication, rate limit (max 10 uploads/hour/user), MIME type (`image/webp`, `image/jpeg`, `image/png`), and file size (<= 5MB).
  3. Client uploads directly to Cloudflare R2 bucket.
  4. Media row inserted into `post_media` with dimensions, size, and mandatory `alt_text`.
  5. Daily cron job cleans orphan uploads older than 24 hours.
- **Acceptance Criteria**:
  - [ ] Direct uploads bypass application server bandwidth.
  - [ ] Images served from custom CDN domain with aggressive caching headers.

### FR-016: Post Card Display & Placeholders
- **Priority**: P0
- **User Roles**: All Users
- **Expected Behaviour**:
  - Renders community name, author username, relative timestamp, title, text excerpt, inline anime tags, vote controls, and comment count.
  - If post status is `removed`, renders placeholder: *"This post was removed by moderators."* (preserves comment tree).
  - If post is marked `is_spoiler`, content is blurred and protected (FR-028).

---

## 6. Rich-Text Editor Module

### FR-017: Restricted Schema Tiptap Editor
- **Priority**: P0
- **User Roles**: Registered User
- **Specification**:
  - **Allowed Nodes**: Document, Paragraph, Heading (level 2, 3 only), BulletList, OrderedList, ListItem, Blockquote, CodeBlock, Custom AnimeTagNode.
  - **Allowed Marks**: Bold, Italic, Strike, Code, Link (protocols: `http`, `https` only), SpoilerMark.
  - **Prohibited**: Raw HTML, embedded videos, `<script>`, `<iframe>`, inline images inside document JSON.
  - **Serialization**: Saved strictly as Tiptap JSON schema. Rendered server-side to static HTML nodes in React Server Components.
- **Acceptance Criteria**:
  - [ ] Pasting arbitrary HTML or script tags strips all unauthorized tags and attributes.
  - [ ] Server renderer outputs accessible, semantic HTML without client-side JS dependency.

---

## 7. Anime Entity Tagging Module

### FR-018: Tag Insertion & Combobox
- **Priority**: P1
- **User Roles**: Registered User (in Editor)
- **Expected Behaviour**:
  1. Typing `@` in editor opens floating combobox.
  2. Debounced search queries `/api/search/anime?q=...`.
  3. Displays list of results with small poster thumbnail, title, and release year.
  4. Selecting item inserts `AnimeTagNode` with attributes: `entityType: 'anime'`, `entityId: malId`, `label: title`.
  5. On post save, tags are validated against `provider_snapshots`. Unresolvable tags downgrade to plain text.
- **Acceptance Criteria**:
  - [ ] Tag node serializes cleanly into document JSON.
  - [ ] Saving post creates corresponding relational entries in `post_tags`.

### FR-019: Inline Tag Pill & Hover Card
- **Priority**: P1
- **User Roles**: All Users
- **Expected Behaviour**:
  1. Inline tag renders as pill button with 20px thumbnail, show title, and subtle cyan border.
  2. Hovering or keyboard focusing tag pill triggers `AnimeHoverCard`.
  3. Hover card fetches `/api/hover/anime/[id]` (cached 24h) and displays: poster, title, score, genres, synopsis snippet, and one-tap `WatchlistControl`.
  4. Dismissible with `Escape` or mouse leave. Viewport-aware positioning avoids overflow.
- **Acceptance Criteria**:
  - [ ] User can add anime to watchlist directly from hover card without navigating away from post.
  - [ ] Works with full keyboard accessibility (Tab, Enter, Escape).

---

## 8. Nested Comments Module

### FR-021: Hierarchical Comment Insertion (PostgreSQL `ltree`)
- **Priority**: P0
- **User Roles**: Registered User
- **Preconditions**: Post published and not locked; depth <= 8.
- **Expected Behaviour**:
  1. User submits comment or reply to parent comment.
  2. Handled via PostgreSQL function `add_comment` with transaction-scoped advisory lock on `post_id`.
  3. Materialized path generated from sequence `comment_path_seq` (e.g., top-level `1`, child `1.2`, grandchild `1.2.5`).
  4. Enforces depth constraint: `depth BETWEEN 1 AND 8`.
  5. Optimistic UI insertion on client; rolls back on failure with toast.
- **Validation Rules**: Content length 1–10,000 characters.
- **Acceptance Criteria**:
  - [ ] 20 concurrent replies to same parent comment execute without race condition or path collision.
  - [ ] Replying to 8th-level comment fails with `DepthExceeded` error.

### FR-022: Thread Assembly & Performance
- **Priority**: P0
- **User Roles**: All Users
- **Expected Behaviour**:
  1. Single indexed query retrieves entire thread ordered by `path`:
     `SELECT ... FROM comments WHERE post_id = $1 AND is_deleted = false ORDER BY path LIMIT 500`.
  2. Server component transforms flat array into nested tree in memory.
  3. Renders visual indent guides using CSS padding; exposes semantic nested `<ul>`/`<li>` structure to screen readers.
- **Performance Budget**: Server processing time < 400ms at p95 for a 1,000-comment thread.
- **Acceptance Criteria**:
  - [ ] 1,000-comment thread loads within 400ms server time.
  - [ ] Visual indentation guides collapse correctly on mobile (max 3 visible levels before "Continue this thread" link).

### FR-023: Comment Deletion & Soft Delete
- **Priority**: P0
- **User Roles**: Comment Author, Moderator
- **Expected Behaviour**: Deleting comment sets `is_deleted = true`. Content replaced with `[deleted]`. Row is retained to prevent orphaning descendant comment branches.

---

## 9. Voting & Ranking Module

### FR-024: Polymorphic Voting Mechanics
- **Priority**: P0
- **User Roles**: Registered User
- **Preconditions**: User signed in; cannot vote on own content.
- **Expected Behaviour**:
  1. User clicks Upvote (+1) or Downvote (-1).
  2. Composite PK `(user_id, post_id)` or `(user_id, comment_id)` guarantees single vote per user.
  3. Clicking active vote removes it (resets to 0). Clicking opposite arrow flips vote.
  4. Optimistic UI update with score delta.
  5. Vote mutation enqueues `recomputeScore` background job.
- **Acceptance Criteria**:
  - [ ] Attempting duplicate vote produces database upsert without duplicate row.
  - [ ] Voting on own post/comment returns `Forbidden` error.

### FR-025: Wilson Score Ranking ("Best")
- **Priority**: P1
- **Specification**: Evaluates lower bound of Wilson Score Interval with 95% confidence level ($z=1.96$):
  $$w = \frac{\hat{p} + \frac{z^2}{2n} - z \sqrt{\frac{\hat{p}(1-\hat{p}) + \frac{z^2}{4n}}{n}}}{1 + \frac{z^2}{n}}$$
  where $n = \text{upvotes} + \text{downvotes}$ and $\hat{p} = \frac{\text{upvotes}}{n}$.
- **Implementation**: PostgreSQL immutable function `wilson_lower(ups, downs)`.
- **Acceptance Criteria**:
  - [ ] A comment with 100 upvotes and 10 downvotes ranks above a comment with 1 upvote and 0 downvotes.
  - [ ] Zero-vote items evaluate to score 0.0.

### FR-026: Hot Ranking ("Hot")
- **Priority**: P1
- **Specification**: Time-decay algorithm based on logarithmic score:
  $$\text{hot\_rank} = \text{sign}(\text{score}) \cdot \log_{10}(\max(|\text{score}|, 1)) + \frac{\text{epoch}}{45000}$$
- **Implementation**: PostgreSQL function `hot_rank(score, created_at)`.
- **Acceptance Criteria**:
  - [ ] High-engagement new posts outrank older high-score posts on `/c/[slug]?sort=hot`.

---

## 10. Spoiler Handling Module

### FR-028: Multi-Level Spoiler Obfuscation
- **Priority**: P1
- **User Roles**: All Users
- **Specification**:
  - Posts: `is_spoiler` boolean flag.
  - Comments: `is_spoiler` boolean flag.
  - Editor: Inline `SpoilerMark`.
- **Security & A11y Requirement**: Spoiler content is obfuscated on server. Content is NOT exposed in DOM or accessibility tree until explicitly clicked.
- **Interaction**: Renders with CSS blur overlay and label *"Contains Spoilers — Click to reveal"*. Clicking removes blur in local client state.
- **Acceptance Criteria**:
  - [ ] Screen readers do not announce spoiler text until reveal button is pressed.
  - [ ] Page reload restores hidden state.

---

## 11. Watchlists Module

### FR-029: Watchlist Tracking & CRUD
- **Priority**: P0
- **User Roles**: Registered User (Self)
- **Preconditions**: Anime exists in database/provider snapshot.
- **Expected Behaviour**:
  1. User can assign status: `watching`, `completed`, `dropped`, `planned`.
  2. User can assign rating score: integer 1–10 (or null).
  3. User can increment/set `episodes_watched`.
  4. Upsert executed against `(user_id, mal_id)` PK.
- **Validation**: `episodes_watched` cannot exceed cached show episode total (unless total is unknown/ongoing).
- **Caching**: Watchlist queries are private and NEVER cached at framework level.
- **Acceptance Criteria**:
  - [ ] Setting status to `completed` automatically prompts to set progress to max episodes.
  - [ ] Watchlist exportable or viewable only by owner at `/u/[username]/watchlist`.

### FR-030: Unified WatchlistControl Component
- **Priority**: P0
- **User Roles**: Registered User
- **Specification**: Shared client component used on Show Detail page, Hover Cards, and Episode Threads. Houses status dropdown, score selector, and episode counter buttons (+/-). All mutations call the same server action `upsertWatchlistEntry`.

---

## 12. Episode Discussion Module

### FR-031: Airing Episode Cache & Thread Generation
- **Priority**: P1
- **User Roles**: All Users, Registered Users
- **Expected Behaviour**:
  1. Shows currently airing have episodes cached in `episodes` table, refreshed daily.
  2. Episode thread route `/anime/[malId]/episodes/[n]` generates on-demand thread linked to composite key `(mal_id, episode_number)` in `episode_threads`.
  3. Creation is idempotent: first visitor creates thread post, subsequent visitors view existing thread.
- **Acceptance Criteria**:
  - [ ] Multiple concurrent visitors accessing an uncreated episode thread trigger exactly one post creation.

---

## 13. Search Module

### FR-033: Full-Text Discussion Search
- **Priority**: P0
- **User Roles**: All Users
- **Expected Behaviour**:
  1. Querying `/search?q=term` searches posts using PostgreSQL generated `tsvector` column with `GIN` index:
     - Title weighted `A`.
     - Body text weighted `B`.
  2. Trigram index (`pg_trgm`) provides fuzzy matching for titles.
  3. Supports filtering by community slug and author.
- **Acceptance Criteria**:
  - [ ] Searching with a slight typo in anime title returns matching posts.
  - [ ] Search query latency < 200ms on database containing 50,000 posts.

---

## 14. Moderation & Trust Module

### FR-035: Content Reporting
- **Priority**: P1
- **User Roles**: Registered User
- **Preconditions**: Signed in; cannot report own content; target exists.
- **Expected Behaviour**:
  1. User opens `ReportDialog` and selects reason: `spam`, `harassment`, `spoilers`, `off_topic`, `illegal`.
  2. System enforces uniqueness on `(reporter_id, target_type, target_id)`.
  3. Confirmation toast displayed without disclosing existing report counts.
- **Acceptance Criteria**:
  - [ ] Submitting duplicate report returns `Conflict` gracefully.

### FR-036: Automated Moderation Thresholds
- **Priority**: P1
- **Specification**: When a post or comment accumulates 3 unique reports from distinct accounts older than 24 hours:
  - System changes target status to `filtered`.
  - Item is removed from public feeds and placed in `/mod` queue.
  - Notification sent to community moderators.
- **Acceptance Criteria**:
  - [ ] 3 valid reports automatically hide post from public view.

### FR-037: Moderator Queue & Action Log
- **Priority**: P1
- **User Roles**: Community Moderator (for community), Admin (global)
- **Expected Behaviour**:
  1. Interface at `/mod` lists pending reports sorted by severity and age.
  2. Actions: `approve` (restores status), `remove` (sets status `removed`), `filter`.
  3. Every decision writes immutable entry to `moderation_actions` and `audit_log` with actor ID, reason, and IP hash.
- **Acceptance Criteria**:
  - [ ] Community moderator cannot action content outside their assigned community.

### FR-038: Account-Level Shadow Filtering
- **Priority**: P1
- **User Roles**: Administrator
- **Specification**: Flag `is_shadowbanned` on `users` table. User experiences normal site operation, but their posts, comments, and votes are filtered server-side from all public queries. Reserved strictly for automated spam bots.

---

## 15. Administration & Monitoring Module

### FR-040: Health & Operational Check
- **Priority**: P0
- **Route**: `GET /api/health`
- **Expected Behaviour**: Verifies PostgreSQL connection, Upstash Redis ping, and circuit breaker status. Returns HTTP 200 `{"status": "healthy"}` or HTTP 503 with degraded component list.

---

## 16. In-App Notifications Module (P2)

### FR-041: Notification Center
- **Priority**: P2
- **User Roles**: Registered User
- **Expected Behaviour**: In-app bell icon with unread badge counter. Alerts for comment replies, username mentions, and moderation decisions. Read state updated via server action. No WebSocket/real-time push required in MVP.

---

## Non-Functional Requirements (NFR)

### Performance & Latency
- **NFR-001**: **LCP Budget**: Largest Contentful Paint must be < 2.5s on a simulated mid-range mobile device on 4G network.
- **NFR-002**: **Thread Rendering Budget**: Server response time for assembling and rendering a 1,000-comment thread must be < 400ms at p95.
- **NFR-003**: **Client Bundle Budget**: First-load JavaScript bundle must not exceed 180 KB compressed for feed pages, and 200 KB compressed for show detail pages.
- **NFR-004**: **Layout Stability**: Cumulative Layout Shift (CLS) must remain < 0.1 across all routes. All posters, cards, and skeletons must declare explicit dimensions.
- **NFR-005**: **Cache Efficiency**: Outbound provider API calls for warm cache visits must be zero.

### Reliability & Resilience
- **NFR-006**: **Zero Provider Downtime Impact**: If the external metadata provider experiences complete outage or returns HTTP 429/500, the platform must serve cached `provider_snapshots` with a visible `DegradedNotice`. No user-facing 500 error page may be rendered due to upstream API failures.
- **NFR-007**: **Database Scalability**: The comment hierarchy must scale to tens of thousands of comments without recursive query penalties, guaranteed by PostgreSQL `ltree` with GiST index.

### Accessibility (a11y)
- **NFR-008**: **WCAG 2.1 AA Compliance**: All typography and color pairs must meet or exceed 4.5:1 contrast for normal text and 3:1 for large text.
- **NFR-009**: **Keyboard Operability**: Every interactive element must be reachable and operable via keyboard alone with an unmistakable focus ring (`--focus-ring: #22D3EE 2px`).
- **NFR-010**: **Screen Reader Support**: Comment trees must expose semantic hierarchy through nested lists; vote counts must be announced as text; spoilers must be masked from the accessibility tree until revealed.
- **NFR-011**: **Zoom & Responsive Adaptation**: Interface must scale to 200% zoom and function at 320px viewport width without horizontal scrolling.
- **NFR-012**: **Reduced Motion**: All animations and transitions must honor `@media (prefers-reduced-motion: reduce)`.

### Security
- **NFR-013**: **Content Sanitization**: All user-submitted Markdown or Tiptap JSON must be sanitized against strict node allowlists. No raw HTML injection is permitted under any circumstance.
- **NFR-014**: **Rate Limiting**: Distributed token bucket in Redis enforces limits: max 5 posts/hour for accounts <24h old, 60 requests/min on public search endpoints.
- **NFR-015**: **Data Protection & Privacy**: Server logs and audit trails must never store plain IP addresses, email addresses, or session tokens. IP addresses are hashed using a rotating salt and retained for 30 days maximum.

---

## Explicitly Excluded Scope (Version 1)

The following items are strictly out of scope for MVP. Any pull request introducing these without an approved ADR will be rejected:
1. Real-time chat rooms or WebSocket infrastructure.
2. Direct messaging (DM) between users.
3. Native e-commerce, checkout, or payment gateway handling.
4. Native video uploading or streaming.
5. Proprietary machine learning recommendation engines.
6. Character-level tagging inside the editor (deferred to P2).
7. Outbound email notifications or digests.
8. Native iOS / Android mobile applications.
9. Localization/internationalization beyond English.
10. Bulk scraping or complete offline mirroring of external anime databases.

