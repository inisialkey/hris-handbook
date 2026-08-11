# CLAUDE.md — HRIS Engineering Handbook Project

## What this repository is

This repository contains the Engineering Handbook for **HRIS**, a multi-tenant enterprise HRIS SaaS for the Indonesian market (business workflows inspired by Mekari Talenta, built on a modern stack). The handbook is generated **file by file** according to `HANDBOOK_SPEC.md` and will become the single source of truth for engineers and AI assistants who implement the product.

## Session rules

1. Start every session by reading `PROGRESS.md`. End every task by updating it.
2. `HANDBOOK_SPEC.md` is the authoritative specification. Read the relevant sections before writing anything.
3. Generate **one large file** (or at most three small files, e.g. ADRs) per task. Never bulk-generate the handbook.
4. Never contradict an anchor document (see Anchors below). To change a decision: supersede its ADR first, then update dependent documents, and record the change in `PROGRESS.md`.
5. Trust the repository, not conversation memory. After `/clear` or a new session, state lives only in the files.

## Hard technology constraints (non-negotiable)

| Layer | Must use | Must NOT use |
|---|---|---|
| Mobile | Flutter (Android + iOS), Clean Architecture feature-first, flutter_bloc (Cubit is the default; Bloc only for justified complex workflows), Drift for all business data, repository pattern, use cases, DI, Result pattern, offline-first | Riverpod, Hive, Flutter Web, SharedPreferences for business data (trivial app settings only) |
| Admin web | Next.js App Router + TypeScript, feature-based architecture, React Query, React Hook Form + Zod, Tailwind + shadcn/ui, TanStack Table, Axios, Server Actions only where appropriate | Pages Router, desktop application (not in V1) |
| Backend | NestJS modular monolith (microservice-ready boundaries), Clean Architecture DDD-inspired, repository pattern, DI, Drizzle ORM, Swagger, JWT + refresh token | Prisma, CQRS without a justifying ADR, raw SQL outside repositories |
| Data / infra | PostgreSQL (shared DB, `tenant_id` isolation), Redis, BullMQ, Firebase Storage, Firebase Cloud Messaging | |

## Decision hierarchy

1. Decided in `HANDBOOK_SPEC.md` §5 → follow it.
2. Listed in §6 (proposed defaults) → follow it once confirmed in Phase 0.
3. Minor gap → decide using industry best practice; log it in `ASSUMPTIONS.md`.
4. Architectural gap → decide; write an ADR with status **Proposed**; flag it for user review in your task report.
5. Business, legal, or commercial unknown → batch questions and ask the user. Never invent regulatory numbers — use the verify marker instead (see Writing standards).

## Writing standards

- Language: English. Keep official Indonesian regulatory terms in Indonesian (PPh 21, BPJS, THR, PKWT, PKWTT, cuti, BPJS Ketenagakerjaan, etc.).
- Money: IDR. Time: store UTC; display in branch timezone (WIB / WITA / WIT).
- Depth over length: every API endpoint has a request schema, response schema, and error codes; every entity has a Drizzle schema definition; every entity with a lifecycle has a Mermaid state diagram; every module has a permission matrix.
- Modules reference the global standards documents and describe **deviations only**. No repeated boilerplate across modules. No filler prose.
- Every regulation-dependent value (tax rate, BPJS cap, statutory leave, overtime multiplier) must carry the marker:
  `> ⚠️ VERIFY: confirm against current Indonesian regulation before implementation.`
- Rationale / alternatives / tradeoffs belong in ADRs only — not inline in every document.

## Anchor documents (authority order — re-read the relevant ones before dependent work)

1. `HANDBOOK_SPEC.md`
2. `docs/adr/` (Accepted status overrides Proposed)
3. `CONTEXT.md` (root — glossary / domain language), `docs/03-standards/naming-conventions.md`, `docs/03-standards/error-catalog.md`, `docs/03-standards/design-system.md` (for any UI Flow work)
4. `docs/04-database/database-conventions.md` and `docs/04-database/core-schema.md`
5. The approved reference module `docs/06-modules/holiday.md` — the structural template for every business module

## Registries (append in the same session that introduces the entry)

- New domain term → `CONTEXT.md` (root glossary)
- New error code → `docs/03-standards/error-catalog.md`
- New assumption → `ASSUMPTIONS.md`

## Grilling sessions (grill-with-docs)

The user may run `/grill-with-docs` to stress-test a plan or a document before approval. For those sessions: the domain glossary is `CONTEXT.md` (root) and decisions live in `docs/adr/`. Any ADR created or edited during a grilling session must follow the `HANDBOOK_SPEC.md` §9.2 template and numbering, and any glossary or ADR change made mid-session must be reflected in `PROGRESS.md` like any other change.

## Design skills (ui-ux-pro-max, frontend-design)

Two design skills are installed and may auto-trigger on UI work. Platform split is strict: **`ui-ux-pro-max` advises the Flutter employee app only; `frontend-design` advises the Next.js admin app only.** Never apply ui-ux-pro-max style/palette/stack recommendations to the admin web, or frontend-design guidance to Flutter. Precedence: the hard constraints in this file and `docs/03-standards/design-system.md` (once written) always override skill suggestions — the skills inform the design-system document; afterwards the document binds. During handbook generation these skills are relevant only to `design-system.md` and module UI Flow sections; their main use comes later, in the implementation repositories.

## Definition of Done

Before marking any file complete in `PROGRESS.md`, run the checklist in `HANDBOOK_SPEC.md` §12. No exceptions.

## Agent skills

### Issue tracker

GitHub Issues on `inisialkey/hris-handbook`, via the `gh` CLI. See `docs/agents/issue-tracker.md`.

One namespace, per `ADR-0025` decision 2: `hris-api`, `hris-admin`, and `hris-mobile` file handbook issues and pull requests **here**, not in their own trackers.

### Triage labels

Default vocabulary — `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` (root glossary) + `docs/adr/`. See `docs/agents/domain.md`.

<!-- rtk-instructions v2 -->
# RTK — token-filtered commands

Prefix a command with `rtk` and its output is filtered before it reaches context;
a command RTK has no filter for passes through unchanged. Failures are not
swallowed, and the unfiltered output is teed to
`~/Library/Application Support/rtk/tee/`.

Use it inside `&&` chains too: `rtk git add . && rtk git commit -m "msg"`.

This is a documentation repository, so the useful filters are the git, GitHub and
search ones:

```bash
rtk git status           # …and log, diff, show, add, commit, push — all subcommands pass through
rtk gh pr view           # …and pr checks, run list, issue list, api
rtk grep <pattern>       # …and ls, read, find — grouped by file/dir
rtk err <cmd>            # errors only, from any command
rtk summary <cmd>        # smart summary of any output
rtk proxy <cmd>          # run unfiltered, for debugging
rtk gain                 # savings stats
```

The handbook checks run as `node scripts/*.mjs`; RTK has no `node` filter, so wrap
them as `rtk err node scripts/handbook-check.mjs` when you only want failures.

Project-local filters live in `.rtk/filters.toml`. The stack-specific command lists
(cargo, docker, kubectl, prisma, pytest, rspec…) were removed — none run here.
`rtk init` re-adds the full list.
<!-- /rtk-instructions -->
