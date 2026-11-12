# Anivio — Product Requirements & Vision

> **Contextualized Social Discovery** — where anime discovery meets community discussion.

This document defines the product vision, target audience, competitive positioning, MVP boundaries, and success metrics for Anivio. It is the authoritative source for *what* we are building and *why*.

- For detailed feature specifications, see [SPEC.md](SPEC.md).
- For technology choices, see [TECHSTACK.md](TECHSTACK.md).
- For locked architectural and design decisions, see [DECISIONS.md](DECISIONS.md).

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Vision & Mission](#vision--mission)
3. [Product Overview](#product-overview)
4. [Product Positioning & USP](#product-positioning--usp)
5. [Core Product Pillars](#core-product-pillars)
6. [Product Principles](#product-principles)
7. [Target Audience](#target-audience)
8. [User Value Proposition](#user-value-proposition)
9. [Competitive Analysis](#competitive-analysis)
10. [MVP Scope](#mvp-scope)
11. [Product Success Metrics](#product-success-metrics)
12. [Business & Monetization](#business--monetization)
13. [Long-term Product Direction](#long-term-product-direction)
14. [Constraints](#constraints)

---

## Problem Statement

Anime fandom is **data-heavy and highly social**, yet no single platform accommodates both needs.

Users today maintain structured data — watchlists, scores, episode progress — on database platforms like MyAnimeList and AniList, which offer comprehensive metadata but rely on outdated, static forums for community interaction. Simultaneously, they turn to Reddit and X for dynamic, algorithmically surfaced discussions, but these platforms lack any integrated anime metadata — a show title is just plain text with no attached score, synopsis, or watchlist action.

**The fundamental problem is the persistent disconnect between consuming structured data and generating social discourse.** Every time a user reads a compelling recommendation on Reddit and wants to add a show to their watchlist, they must context-switch to a separate platform. Every time they finish an episode on their tracker and want to discuss it, they must navigate elsewhere. This cross-platform friction is the core pain Anivio exists to solve.

---

## Vision & Mission

### Vision

Bridge the gap between discourse and database — a platform where every anime mentioned is a **live object** with its own score, artwork, and one-tap watchlist action, organized into communities, with threaded comments ranked by statistical confidence.

### Mission

Create a unified ecosystem where **data fuels discussion** and **discussion drives discovery**.

---

## Product Overview

Anivio is a centralized anime community platform — a **"contextualized social discovery engine."** It unifies structured anime metadata with dynamic, community-driven social discussion, eliminating the friction of cross-platform context switching between database platforms (MAL, AniList) and social platforms (Reddit).

When a user creates a post about an anime, the text is semantically linked to the underlying data object. The platform dynamically injects the show's score, genre tags, and watchlist toggle alongside the user's content. The result: **read a review → immediately track the show**, with zero friction.

---

## Product Positioning & USP

### Positioning

**"Contextualized Social Discovery"** — unlike Reddit (where an anime title is plain text) or MAL (where community means static forums), Anivio treats anime as **native interactive entities** within the social feed.

### Unique Selling Proposition

When a user creates a post about an anime, the text is semantically linked to the underlying data object. The platform dynamically injects the show's score, genre tags, and watchlist toggle alongside the user's content. Discovery friction is eliminated: **read a review → immediately track the show.**

This is not a feature bolted on — it is the architectural foundation. Every post, comment, and feed item is metadata-aware. See [SPEC.md](SPEC.md) for the detailed inline anime tag card specification, and [DECISIONS.md](DECISIONS.md) for the rationale behind treating anime as first-class entities.

---

## Core Product Pillars

| # | Pillar | Description |
|---|--------|-------------|
| 1 | **Contextualized Metadata** | Anime entities are first-class objects in discussion — not plain text, but live cards with scores, artwork, and actions. |
| 2 | **Community-Driven Discussion** | Reddit-style threaded feeds with Wilson Score confidence-based ranking (Hot / Best / New). |
| 3 | **Unified Tracking** | Watchlist management and social discussion in one place — no context-switching. |
| 4 | **Provider Resilience** | Graceful degradation when external data sources (Tenrai, AniList) are unavailable. See [TECHSTACK.md](TECHSTACK.md) for the metadata adapter layer design. |

---

## Product Principles

> [!IMPORTANT]
> These principles govern every UI and UX decision. They are non-negotiable for MVP.

1. **Colour is information, not decoration** — colour conveys meaning (status, category, emphasis), never aesthetics alone.
2. **Posters are the only loud element** — the interface stays quiet; anime artwork is the visual anchor.
3. **One card, one idea** — each card in a feed communicates a single concept.
4. **Density with air** — information-dense layouts, but rhythmically separated so nothing feels cramped.
5. **Never lie about state** — pending, failed, stale, and empty states all look visually distinct and honest.

---

## Target Audience

### Primary Personas

#### 1. The Seasonal Tracker

- **Profile**: Actively watches 5–15 shows per season.
- **Core Need**: Immediate post-broadcast discussion and real-time progress tracking.
- **Current Behaviour**: Weekly episode threads on Reddit → update progress on MAL/AniList → return to Reddit.
- **Anivio Value**: One place to discuss episodes *and* track progress, with inline metadata on every post.

#### 2. The Archival Reviewer

- **Profile**: Maintains extensive watchlists (hundreds of shows). Writes long-form critical reviews.
- **Core Need**: Structured review authoring with audience reach and curated recommendation lists.
- **Current Behaviour**: Reviews on MAL/AniList with limited social reach → promotes on Reddit/X for visibility.
- **Anivio Value**: Structured reviews with live metadata context, community engagement, and persistent discoverability.

### Secondary Persona

#### 3. The Casual Discoverer

- **Profile**: Watches sporadically. Relies on consensus and recommendations to decide what to watch.
- **Core Need**: Low-effort discovery of quality shows through community curation.
- **Current Behaviour**: Browses Reddit frontpage or asks friends → manually searches for shows on a separate tracker.
- **Anivio Value**: Algorithmically surfaced Hot feeds with instant one-click watchlist access — discover and save in one tap.

### Tertiary Persona *(POST-MVP)*

#### 4. The Merchandise Collector

> [!NOTE]
> **Status: PROPOSED / FUTURE** — This persona is tied to monetization phases (see [Business & Monetization](#business--monetization)) and is explicitly out of MVP scope.

- **Profile**: Invested in physical media, figures, and collectibles.
- **Core Need**: Contextually relevant merchandise discovery tied to shows they love.
- **Anivio Value**: Contextual affiliate links on anime metadata cards ("Buy Merch" actions).

---

## User Value Proposition

| Persona | Value Statement |
|---------|-----------------|
| Seasonal Trackers | One place to discuss episodes **and** track progress — no more switching between Reddit and MAL. |
| Archival Reviewers | Structured reviews with live metadata context — scores, genres, and watchlist actions embedded alongside your critique. |
| Casual Discoverers | Algorithmically surfaced content with instant watchlist access — find a show, save it, done. |

---

## Competitive Analysis

| Competitor | Strengths | Limitations | Anivio Differentiation |
|------------|-----------|-------------|------------------------|
| **Reddit** (r/anime) | Advanced algorithmic sorting (Wilson/Hot), massive user base, dynamic discussion culture | No native anime metadata, watchlists, or persistent reviews. Discussions are ephemeral and disconnected from data. | Integrate structured metadata directly into feeds. One-click watchlist from within any discussion thread. |
| **MyAnimeList** | Largest, most comprehensive anime database. Official API. Industry standard for tracking. | Outdated UI/UX, static forums with minimal engagement. Official API limits commercial use. | Modern dynamic feed powered by anime data. Community-first with data as the support layer, not the other way around. |
| **AniList** | Modern UI/UX, powerful GraphQL API, activity feeds. | Rate limited to 30 req/min. Forums are rigid and treated as secondary to tracking. | Make community interaction the primary experience, with data as the contextual support layer. |
| **Anime-Planet** | Internal recommendation engine, integrated streaming links, legal streaming focus. | Smaller community. Less focus on user-generated episode debate and threaded discussion. | Deep user-generated content: rich nested threads (8 levels), dynamic discussions, community-driven discovery. |

### Competitive Positioning Map

```
                    Strong Metadata / Tracking
                            │
              MAL ●         │         ● AniList
                            │
                            │      ● Anivio
   Weak Community ──────────┼────────── Strong Community
                            │
                            │
         Anime-Planet ●     │         ● Reddit (r/anime)
                            │
                    Weak Metadata / Tracking
```

Anivio occupies the **upper-right quadrant**: strong metadata *and* strong community — a space no existing platform fills.

---

## MVP Scope

> [!IMPORTANT]
> This scope is locked for Version 1. See [DECISIONS.md](DECISIONS.md) for the rationale behind specific inclusions and exclusions.

### ✅ In Scope (Version 1) — CONFIRMED

| Area | Features |
|------|----------|
| **Authentication** | Google OAuth, Discord OAuth, unique username selection, public profile page |
| **Anime Discovery** | Current season browser, top anime list, search, show detail page with episodes and characters |
| **Communities** | Create, browse, join communities; per-community feed |
| **Posts** | Text posts with optional inline anime tags, spoiler flag, up to 4 image attachments |
| **Comments** | Nested to 8 levels (ltree), collapsible threads, spoiler flags on individual comments |
| **Voting** | Up/down on posts and comments; Hot / Best / New feed algorithms |
| **Watchlist** | Status tracking (watching / completed / dropped / plan-to-watch), user score, episode progress |
| **Moderation** | User reporting, threshold-based actions, content removal, account-level filtering, audit trail |
| **Search** | Full-text search across posts and comments |
| **Legal** | Terms of service, privacy policy, content policy, DMCA/takedown contact |

For detailed feature specifications, acceptance criteria, and interaction flows, see [SPEC.md](SPEC.md).

### ❌ Explicitly Out of Scope

The following are intentionally excluded from MVP to maintain focus and feasibility:

- Real-time chat / instant messaging
- Direct messages (DMs)
- Native store / payment processing
- Video hosting
- Custom recommendation ML models
- Character-level tagging (deferred to post-MVP; see [Long-term Direction](#long-term-product-direction))
- Email notifications
- Mobile applications (native iOS/Android)
- Localization beyond English
- Bulk mirroring of anime catalogue

---

## Product Success Metrics

> [!NOTE]
> These metrics define "done" for the MVP launch. They are measured before declaring the product ready for public beta.

| Criterion | Target | Measurement Method |
|-----------|--------|-------------------|
| **Working Product** | All P0 features complete; all acceptance tests passing | Automated test suite + manual walkthrough |
| **Usable by Strangers** | 10 people complete sign-up → first comment without assistance | Unmoderated usability session with 10 testers |
| **Performance** | Feed and show pages render in < 2.5s on simulated mid-range phone / 4G | Lighthouse mobile audit on production URL |
| **Reliability** | Site renders correctly with anime metadata provider fully unavailable | Deliberate provider failure test in staging environment |
| **Community Traction** | 50 beta testers; 5+ communities with real discussion; at least one thread reaching 5 nesting levels deep | Database counts after 4 weeks live |

---

## Business & Monetization

> [!WARNING]
> **Status: PROPOSED / FUTURE** — All monetization is post-MVP. No revenue features are included in Version 1. These plans are directional and subject to validation.

### Phased Monetization Roadmap

| Phase | Strategy | Details |
|-------|----------|---------|
| **Phase 1** (Post-Launch) | Contextual affiliate links | "Buy Merch" button on anime metadata cards. Partners: Tokyo Otaku Mode (5–8%+ commission), Crunchyroll Store. |
| **Phase 2** (After significant MAU) | Programmatic display ads | Google AdSense integration in non-intrusive placements. |
| **Phase 3** (Established community) | Premium subscription tier | Cosmetic upgrades: custom avatars, profile badges, GIF profile pictures. No paywalled content. |

### Compliance Requirements

- All affiliate and sponsored content **must** display visible "Sponsored" or "Affiliate" badges (FTC compliance).
- Monetization activation requires upgrading hosting from Vercel Hobby to Vercel Pro plan.

### Revenue Prerequisites

Monetization cannot begin until:
1. Community traction metrics are met (see [Success Metrics](#product-success-metrics)).
2. Legal entity or payment structure is established (currently: no company, no payments infrastructure).

---

## Long-term Product Direction

> [!NOTE]
> **Status: PROPOSED — NOT COMMITTED.** These items represent potential future directions based on user feedback and platform growth. None are scheduled or promised.

| Feature | Notes |
|---------|-------|
| Character-level tagging | Deferred from MVP. Would allow tagging specific characters in posts, not just shows. |
| Email notifications | Digest and real-time notification emails for replies, votes, and community activity. |
| Premium subscription tier | See [Phase 3 monetization](#phased-monetization-roadmap). Cosmetics only — no content gates. |
| Affiliate monetization | See [Phase 1 monetization](#phased-monetization-roadmap). Contextual, clearly labelled. |
| Custom recommendation engine | ML-based recommendations beyond community-driven Hot/Best feeds. |
| Localization | Multi-language support beyond English. Requires community translation infrastructure. |
| Mobile applications | Native iOS/Android apps. Would require evaluating React Native or similar. |

---

## Constraints

These constraints shape every product and technical decision. They are not limitations to overcome — they are the boundaries within which Anivio must succeed.

| Constraint | Impact |
|------------|--------|
| **Solo developer** | ~18 focused hours/week, alongside a degree programme. Every feature must justify its implementation cost. |
| **Zero budget** | No hosting costs beyond free tiers until monetized. Target: up to ~$50/month once revenue exists. |
| **No company** | No legal entity, no payment processing, no staff. Monetization phases require establishing these. |
| **English only** | Single-language launch. Localization is a post-MVP concern. |

These constraints directly informed technology choices (see [TECHSTACK.md](TECHSTACK.md)) and architectural decisions (see [DECISIONS.md](DECISIONS.md)).

---

*Last updated: November 2026*
