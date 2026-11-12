# Anivio — Contributing & Engineering Standards

> **Single Source of Truth (SSOT) for Git Workflow, Code Standards & Review Checklists**  
> Status: **LOCKED BASELINE (Version 1.0)**  
> Last Updated: 12 November 2026

This document specifies the engineering practices, Git branching workflows, naming conventions, TypeScript standards, and code review criteria for all contributors and AI coding agents working on the Anivio codebase.

---

## 1. Git Workflow & Branching Strategy

Anivio follows a structured trunk-based Git workflow with short-lived feature branches:

- **`main` Branch**: Production-ready code only. Direct commits to `main` are permanently disabled via branch protection. Merges require all CI quality checks to pass.
- **Feature Branches**: Branch from `main` and name using standard prefixes:
  - `feat/feature-name` (e.g. `feat/tiptap-tag-node`)
  - `fix/bug-description` (e.g. `fix/comment-depth-cap`)
  - `chore/task-name` (e.g. `chore/upgrade-drizzle`)
  - `docs/doc-update` (e.g. `docs/api-contracts`)
  - `perf/optimization` (e.g. `perf/gist-index-tuning`)

### Commit Conventions
- Use conventional commits format: `<type>(<scope>): <concise description>`
- Examples:
  - `feat(discussions): implement optimistic voting action`
  - `fix(comments): enforce advisory lock on post comment insertion`
  - `docs(spec): document watchlist episode tracking edge case`
- **Rule**: One logical change per commit. Commit messages must state the *why*, not merely the *what*.

### Pull Request Lifecycle
1. Open PR as a **Draft** early to share diff progress.
2. Ensure automated CI checks pass: typecheck, lint, unit tests, migration drift, and bundle size checks.
3. Every PR that modifies business logic, API contracts, or database schema must update the relevant documentation in `docs/` in the same pull request.

---

## 2. Naming Conventions Reference Table

| Category | Convention | Pattern / Rule | Example |
|---|---|---|---|
| **Files & Folders** | `kebab-case` | Grouped by domain, not file type | `src/modules/discussions/create-post.ts` |
| **React Components** | `PascalCase` | One component per file, matches filename | `CommentNode.tsx`, `AnimeCard.tsx` |
| **React Hooks** | `camelCase` | Prefixed with `use` | `useOptimisticVote.ts` |
| **Server Actions** | `camelCase` | Verb-first action name | `createComment`, `upsertWatchlistEntry` |
| **Database Tables** | `snake_case` | Plural nouns; join tables alphabetical | `posts`, `post_votes`, `community_members` |
| **Database Columns** | `snake_case` | Explicit descriptors, no abbreviations | `is_shadowbanned`, `wilson_lower` |
| **Environment Vars** | `SCREAMING_SNAKE`| Prefixed with service domain | `AUTH_GOOGLE_ID`, `R2_BUCKET` |
| **SQL Migrations** | `number_name.sql` | 4-digit sequential prefix, forward-only | `0005_comments.sql` |
| **Error Codes** | `PascalCase` | Discriminated string literals, stable | `DepthExceeded`, `SlugTaken` |

---

## 3. TypeScript & Code Standards

- **Strict Mode**: TypeScript is configured with `"strict": true`, `"noImplicitAny": true`, and `"strictNullChecks": true`.
- **Explicit Returns**: Server actions and shared domain queries must declare explicit return types.
- **Zero `any` Policy**: The use of `any` is forbidden. Use `unknown` with type narrowing or generic constraints.
- **Zod Schema Sharing**: Validation schemas must be exported from domain modules and shared between client forms and server actions.

---

## 4. Component & Architecture Rules

### 4.1 Server vs. Client Components
- **Server by Default**: Components are Server Components unless user interactivity (`onClick`, `onChange`), browser APIs, or optimistic state (`useOptimistic`) are required.
- **Isolate Client Islands**: Keep client components small and placed at the leaves of the tree (e.g. `VoteControl.tsx`, `JoinButton.tsx`). Never mark entire page layouts as `'use client'`.

### 4.2 Data Access & Mutations
- **No Direct DB Queries in Client Components**: All database operations occur in Server Components or Server Actions.
- **Transaction Integrity**: Mutations affecting multiple tables must wrap queries inside `db.transaction(async (tx) => ...)`.
- **Parameterization**: Raw SQL fragments must consume variables through parameterized arguments (`sql\`...\``), never string interpolation.

---

## 5. Documentation Maintenance Policy (Rule of Documentation)

Because this documentation suite serves as the Single Source of Truth:
- **No Undocumented Feature**: Code changes that alter user-visible behaviour, routes, or database schemas without an accompanying documentation change are rejected.
- **Traceability**: New requirements must reference their parent IDs (`FR-xxx`, `NFR-xxx`, `ADR-xxx`).
- **Conflict Prevention**: If two documents diverge, the authoritative document designated in `docs/README.md` must be followed, and the contradiction resolved immediately.

---

## 6. Pre-Merge Code Review Checklist

Before approving any pull request, verify:

- [ ] Does the code adhere to the established directory structure in [STRUCTURE.md](STRUCTURE.md)?
- [ ] Are all database operations non-destructive and backward-compatible per [DEPLOYMENT.md](DEPLOYMENT.md)?
- [ ] Are all inputs validated using shared Zod schemas per [SPEC.md](SPEC.md)?
- [ ] Are all touch targets >= 44x44px and focus rings visible per [DESIGN.md](DESIGN.md)?
- [ ] Does the change maintain the performance budget (bundle < 180 KB) per [TESTING.md](TESTING.md)?
- [ ] If an architectural change was introduced, is an ADR recorded in [DECISIONS.md](DECISIONS.md)?
- [ ] Have the corresponding documentation files in `docs/` been updated in this pull request?

