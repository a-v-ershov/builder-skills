# builder-skills

**Take a raw idea to a shipped release with [Claude Code](https://claude.com/claude-code) — one
reviewed step at a time, so the AI builds the *right* thing and proves it works instead of just
saying so.**

Three gated pipelines — **spec → build → release** — installed as a plugin into any project from
GitHub. Every skill replies in the language you write to it.

---

## The 3 problems it fixes

1. **AI builds the wrong thing, fast** — it jumps from a one-line prompt straight to code, so you ship a polished version of an unvalidated idea and find out too late.
2. **AI grades its own homework** — the same agent writes the feature, declares *"it works,"* and writes tests that pass because it wrote them to pass.
3. **Quality and decisions vanish** — security holes, slow paths, and the reasoning behind choices never show up in a green unit test and are lost between sessions.

## What you get

1. **Three pipelines, one command each** — call `create-project-spec`, `build-product`, or `release-product` and one meta-skill conducts every phase for you.
2. **It interviews you before it builds** — a discovery grill pulls the maximum context out of your head, one question at a time, always with a recommended answer.
3. **A separate agent always checks the work** — nothing self-certifies: a reviewer on every spec phase, plus a fresh verifier that writes adversarial tests and is hook-blocked from touching the code.
4. **A real dev loop + release audits that hunt the AI's own bugs** — an enforced quality gate locally, then independent security / performance / quality / accessibility / doc audits before you ship.

**Plus:** replies in your language · reverse-engineers an existing codebase into a spec (brownfield) ·
leaves committed project memory — docs, ADRs, backlog, a `CLAUDE.md` map — that doesn't rot · ships as
a plain Claude Code plugin, no extra runtime or MCP server.

---

## How it works

Three pipelines, each a thin **orchestrator** that conducts focused sub-skills; every sub-skill also
runs on its own.

### 1 — Spec: idea → buildable spec

`create-project-spec` runs seven persona-driven phases; each researches its claims (source-cited),
drafts, is checked by an adversarial reviewer, and emits a research doc + a short human summary under
`docs/project-spec/`.

| Step | Skill | Persona | Produces |
|------|-------|---------|----------|
| 1 | `gather-context` | Discovery interviewer | `project-brief` — interviews you until you share the same understanding |
| 2 | `validate-idea` | Founder-turned-investor | `idea-validation` — KILL / SHRINK / forcing questions |
| 3 | `define-product-requirements` | Product manager | `product-requirements` — the full feature set, each with testable criteria |
| 4 | `create-user-flows` | Product designer | `user-flows` — how users move through the product |
| 5 | `define-design-decisions` | Design-system lead | `design-decisions` — direction, not mockups; the bridge to tech |
| 6 | `design-architecture` | Software architect | `architecture` + ADRs — quality scenarios first, then components + tech |
| 7 | `design-dev-architecture` | DX / platform engineer | `dev-architecture` + ADRs — the local inner loop and AI tooling |

> **Brownfield:** point it at existing code (`project_type: existing`) and `map-codebase` runs first
> to chart the as-is facts; every phase then reconstructs a *target* spec and logs the drift.

### 2 — Build: spec → working software

`build-product` turns the spec into code sequentially — one task at a time, single working tree, no
parallelism — mutating the real repo.

| Step | Skill | Role |
|------|-------|------|
| 1 | `setup-dev-environment` | Scaffolds the repo, brings up the one-command stack, stands up the enforced quality gate (`make check` + hooks) |
| — | `create-design-system` | *(UI)* makes the design direction concrete as a committed `DESIGN.md` you pick from rendered candidates |
| 2 | `plan-development` | Emits a kanban backlog — one file per task, where `blocked_by` *is* the dependency graph |
| 3 | `implement-feature` | Builds one task and gets the quality gate green |
| 4 | `verify-feature` | A **fresh, separate agent** that authors adversarial tests, drives the real stack, and proves an observable outcome |

It picks the lowest-id ready task, builds it, verifies it in a separate agent (bounded — at the cap a
task escalates to `needs_human`), and on a green gate makes a checkpoint commit. `generate-mockups`
renders UI options on demand; `propagate-changes` walks a later spec edit forward into the backlog.

### 3 — Release: working software → cut release

`release-product` proves the cross-cutting properties no single task could, then ships; the audits are
read-only, so they fan out in **parallel** and file findings as rework — they never fix code themselves.

| Skill | Proves against |
|-------|----------------|
| `audit-security` | the STRIDE-lite threat model — secrets, authz, injection, the lethal trifecta |
| `audit-performance` | the quality-attribute scenarios — measured p95, throughput, N+1, cost |
| `audit-product` | the user flows, end-to-end and cross-feature |
| `audit-code-health` | duplication, dead code, suppression debt, test-suite quality (mutation) |
| `audit-accessibility` | the WCAG target — keyboard, focus, screen-reader, contrast |
| `cut-release` | clean tree + no open 🔴 → docs, version, changelog, tag, commit, PR (always confirmed; **stops before prod deploy**) |

A blocker becomes a rework task, fixed by a `build-product` run, then **re-audited to confirm it
closed**.

---

## How it compares

Most tools cover one slice of the arc; builder-skills' bet is the *full* arc plus a separate adversary
at every stage.

| Tool | What it is | Where builder-skills differs |
|------|-----------|------------------------------|
| **[GitHub Spec Kit](https://github.com/github/spec-kit)** | Spec → Plan → Tasks → Implement; templates, agent-agnostic | It stops at implement — no independent verification, no release audits, no built-in research or adversarial review. |
| **[BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)** | Agentic-agile framework, 12+ specialist agents | Plans and builds, but has no read-only release-audit phase, no harness-enforced verifier, no brownfield reverse-engineering. |
| **[Task Master](https://github.com/eyaltoledano/claude-task-master)** | PRD → task decomposition; an MCP project manager | Task management only — it doesn't validate, research, design, or audit; needs an MCP server running. |
| **[gstack](https://github.com/garrytan/gstack)** | 23 role skills for the per-PR sprint loop | Great on an existing codebase's review/ship loop, but has no idea→spec generator and no spec reconstruction. *(Our design reference.)* |
| **[SuperClaude](https://github.com/SuperClaude-Org/SuperClaude_Framework)** | ~30 commands + persona agents injected into `CLAUDE.md` | An à-la-carte command toolbox, not a sequenced, gated pipeline with research, review, and a writer/reviewer split baked in. |

---

## Install

In any project, inside Claude Code:

```
/plugin marketplace add https://github.com/a-v-ershov/builder-skills.git
/plugin install builder-skills@builder-skills
```

Skills appear namespaced as `builder-skills:<skill>`. Start a project with
**`/builder-skills:create-project-spec`** and answer the three setup questions — or run any single
skill on its own.

## Update

An install only picks up changes once the plugin's `version` is **bumped**. To update a project that
has it installed (a restart applies the update):

```
/plugin marketplace update builder-skills
/plugin update builder-skills@builder-skills
```

Or turn on auto-update once: `/plugin` → **Marketplaces** → `builder-skills` → **Enable auto-update**.

## Develop

This repo is both the marketplace and the plugin it ships — collapsed into the repo root
(`.claude-plugin/` + `skills/` + `agents/`), so the marketplace uses `source: "./"`. Validate after
editing, and test locally from a path:

```
claude plugin validate .
/plugin marketplace add ./
```

The `.githooks/commit-msg` hook blocks AI-attribution trailers in commit messages — enable it once per
clone with `git config core.hooksPath .githooks`. See [`CLAUDE.md`](CLAUDE.md) for the full design
conventions.
