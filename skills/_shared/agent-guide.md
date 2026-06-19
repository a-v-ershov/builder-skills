# Project documentation map (agent guide) — shared

A small, navigational section that lives in the **target project's root `CLAUDE.md`** (the product
repo being specced/built — not this plugin). It orients any coding agent to where the project's own
documentation lives — the spec, the backlog, the setup contract, and the release audits — and the
order to read it in before touching code. It is a **map, not a copy**: it points at the artifacts, it
never restates them.

This is shared methodology: several skills write or refresh the **same** section, each at a natural
moment, all from this one spec — so the behavior is defined once and never duplicated.

## Who writes / refreshes it, and when

- **`create-project-spec`** — *seeds* it at the start of a run (a skeleton describing the spec to
  come) and *finalizes* it at the end (pointing at the real `.buildloop/project-spec/` artifacts).
- **`setup-dev-environment`** — writes/refreshes it inside the project `CLAUDE.md` it scaffolds in
  the build phase, next to its stack notes + commands (which live **outside** the markers).
- **`plan-development`** — refreshes it after writing the backlog, so `.buildloop/build-plan/` shows up.
- **`release-product`** — refreshes it after the release audits run (it always runs in the release
  phase, so this is the release phase's primary refresh), so `.buildloop/release/` shows up.
- **`cut-release`** — refreshes it when finalizing a standalone cut, keeping `.buildloop/release/` current.

Any of these can also run standalone. The operation is **idempotent** — re-running re-renders the
block in place and never duplicates it.

## Marker discipline (non-destructive — read carefully)

The section is delimited by HTML-comment markers and is the ONLY thing these writers touch:

```
<!-- buildloop:project-map:start -->
…rendered map…
<!-- buildloop:project-map:end -->
```

- **Markers present** → replace everything between them; leave the rest of `CLAUDE.md` untouched.
- **No markers but the file exists** → append the block after the top-of-file title/intro; never
  edit, reorder, or delete the user's existing content.
- **No `CLAUDE.md`** → create it with a one-line title + the block.
- Everything **outside** the markers — the user's own notes, `setup-dev-environment`'s
  stack/commands — is owned by someone else. Never rewrite it. (No `.bak` is needed when you only
  replace between the markers.)

## What to map (scan, mark present vs planned)

Scan `.buildloop/project-spec/`, `.buildloop/build-plan/`, `.buildloop/project-setup/`, `.buildloop/release/`, and the **root
`DESIGN.md`** (UI projects). Emit one row per known artifact, pointing at it. If a file is not there yet (e.g. an early seed before the spec exists),
still list it but mark it **planned** — so the map describes the intended shape from day one. Never
invent artifacts the pipeline does not produce.

Known artifacts (the `.research.md` files are the depth; each has a short `.summary.md` human pair):

- `.buildloop/project-spec/summary.md` — combined human summary (read first).
- `.buildloop/project-spec/codebase-map.research.md` — *(existing projects only)* the as-is map reverse-engineered from the code (structure, stack, domain, surfaces) — what's built, before the TARGET spec.
- `.buildloop/project-spec/project-brief.research.md` — the discovery brief (the user's original intent, scope, constraints).
- `.buildloop/project-spec/idea-validation.research.md` — why this exists (validation).
- `.buildloop/project-spec/product-requirements.research.md` — features, acceptance criteria, domain model.
- `.buildloop/project-spec/user-flows.research.md` — the user flows.
- `.buildloop/project-spec/design-decisions.research.md` — design direction.
- `.buildloop/project-spec/architecture.research.md` (+ `adr/`) — system architecture + decisions.
- `.buildloop/project-spec/dev-architecture.research.md` — inner loop (local run, testing, AI tooling).
- `.buildloop/build-plan/board.md` (+ `tasks/*.md`) — the backlog (what to build next).
- `.buildloop/project-setup/verification.md` — run / drive / prove commands.
- `.buildloop/project-setup/setup-log.md` — what the env provides + manual TODOs.
- `DESIGN.md` *(repo root, UI projects)* — the committed design system (tokens + rules) UI code is built against.
- `.buildloop/project-setup/design-system.md` — *(UI projects)* how the design system was chosen (the *why* behind `DESIGN.md`).
- `.buildloop/release/release-summary.md` — release readiness + what shipped (the verdict).
- `.buildloop/release/*-audit.md` — per-audit findings (security, performance, product, code-health, accessibility) — the audit trail.

## Rendered template (emit this between the markers)

Set each `Status` from what exists at write time: `✓` present · `◦` planned. Keep the block short —
it is a map, not a summary. Drop rows for trees that will never exist if you know they won't (rare).

```markdown
<!-- buildloop:project-map:start -->
## Project documentation map

This project is specced and planned with Buildloop. The docs under `.buildloop/` are the source of
truth for *what* to build and *why* — read them before changing code, and flag (don't silently
absorb) any place where the code and the docs disagree; propagate real changes with
`propagate-changes`.

**Read in this order before implementing:**
1. `.buildloop/project-spec/summary.md` — the whole project in one read.
2. The task you're on under `.buildloop/build-plan/` — its `## Description`, acceptance criteria, `traces_to`.
3. The spec sections it traces to (the rows below).
4. `.buildloop/project-setup/verification.md` — how to run it and prove the change works.
5. For UI work, the root `DESIGN.md` — the design system (tokens + rules) to build against.

**Map** — Status: `✓` present · `◦` planned

| Status | Doc | What it is |
|--------|-----|------------|
| ✓ | `.buildloop/project-spec/summary.md` | Combined human summary — start here |
| ✓ | `.buildloop/project-spec/product-requirements.research.md` | Features, acceptance criteria, domain model |
| ✓ | `.buildloop/project-spec/user-flows.research.md` | User flows |
| ✓ | `.buildloop/project-spec/design-decisions.research.md` | Design direction |
| ✓ | `.buildloop/project-spec/architecture.research.md` (+ `adr/`) | System architecture + decisions |
| ✓ | `.buildloop/project-spec/dev-architecture.research.md` | Local run, testing, AI tooling |
| ◦ | `.buildloop/build-plan/board.md` (+ `tasks/*.md`) | Backlog — what to build next |
| ◦ | `.buildloop/project-setup/verification.md` | Run / drive / prove commands |
| ◦ | `DESIGN.md` (root) | Design system — tokens + rules UI is built against *(UI projects)* |
| ◦ | `.buildloop/release/release-summary.md` | Release readiness + what shipped |
| ◦ | `.buildloop/release/*-audit.md` | Per-audit findings (the audit trail) |
<!-- buildloop:project-map:end -->
```

## Language

Render the prose in the user's language, like every skill — but keep file paths, identifiers, the
markers, and the acceptance keywords verbatim. Never translate them.
