# Anivio — Project Changelog

All notable changes to the Anivio project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0-docs] - 2026-11-12

### Added
- Established the comprehensive Single Source of Truth (SSOT) documentation suite across 16 interconnected Markdown specifications in `docs/`:
  - `README.md` & `docs/README.md`: Project charter, architectural map, and documentation index.
  - `docs/PRODUCT.md`: Product vision, target personas, competitive differentiation, and business goals.
  - `docs/SPEC.md`: Exhaustive functional requirements (`FR-001` through `FR-041`) and non-functional requirements (`NFR-001` through `NFR-015`).
  - `docs/DESIGN.md`: Complete UI/UX design tokens, typography, 30 component specifications, 10 page layouts, and 5 state patterns.
  - `docs/WORKFLOW.md`: Step-by-step user interaction flows and Mermaid sequence diagrams (`WF-001` through `WF-018`).
  - `docs/STRUCTURE.md`: Target directory layout, module boundaries, import restrictions, and server/client component matrix.
  - `docs/TECHSTACK.md`: Pinned technology stack definitions, evaluation criteria, and rejected alternatives.
  - `docs/DATABASE.md`: Authoritative relational ERD, DDL schemas, PostgreSQL `ltree` comment hierarchy function, Wilson Score SQL, and migration plan (0001–0010).
  - `docs/API.md`: Server Action specifications, Route Handler contracts, external anime adapter architecture, and error taxonomy.
  - `docs/SECURITY.md`: Authentication controls, RBAC permission matrix, XSS/CSRF defenses, R2 upload security, and moderation tools.
  - `docs/TESTING.md`: Layered test suite plan (Vitest unit/integration, Playwright E2E journeys, failure injection scenarios).
  - `docs/DEPLOYMENT.md`: Infrastructure topology (Vercel, Neon, Upstash, R2), boot-time validated environment variable schema, and backup policies.
  - `docs/ROADMAP.md`: Ten sequential implementation milestones (M0 through M9) with task checklists and definitions of done.
  - `docs/DECISIONS.md`: Formal Architecture Decision Records (ADR-001 through ADR-012), Open Decisions register, and resolution of source document conflicts.
  - `docs/CONTRIBUTING.md`: Git branching conventions, naming standards, TypeScript rules, and code review checklists.
  - `docs/CHANGELOG.md`: Chronological project history tracking baseline.

### Changed
- Resolved source document contradictions between initial architecture plan and build planning document:
  - Formally locked **Tenrai API v1 + AniList Fallback + Snapshots** as the metadata strategy (superseding unmaintained Jikan v4).
  - Formally locked comment nesting depth at **8 levels** (superseding proposed 10 levels) to guarantee mobile UI stability.
  - Formally locked three-tier caching model (`'use cache'`, Redis token bucket, PostgreSQL snapshots).
  - Formally excluded WebSockets and synchronous chat in favor of asynchronous threaded discussions.

---

## Guidelines for Future Entries

When submitting pull requests that introduce code or architectural modifications:
1. Record changes under the unreleased heading `## [Unreleased]`.
2. Group entries into the standard categories:
   - `Added` for new features or capabilities.
   - `Changed` for modifications to existing behavior.
   - `Fixed` for bug corrections.
   - `Removed` for deprecated or removed functionality.
   - `Security` for vulnerability patches and hardening improvements.
3. Every changelog entry must reference its corresponding task or issue ID (e.g. `TASK-015`).

