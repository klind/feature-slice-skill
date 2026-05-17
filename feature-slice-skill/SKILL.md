---
name: feature-slice-skill
description: A 4-phase workflow for adding features to existing codebases (brownfield or greenfield) one vertical slice at a time, with the strict discipline of touching as little surrounding code as possible. The core rule — add, don't disturb — protects existing code from collateral damage by preferring new files over edits, surgical diffs over refactors, and end-to-end slices over horizontal layer-by-layer builds. Phase 1 — relentlessly clarify and lock scope via the AskUserQuestion tool before any code is written. Phase 2 — rewrite the plan into vertical end-to-end slices (model + DB + API + UI + tests + integration) that lean additive and list new vs. modified files explicitly. Phase 3 — implement one slice at a time with baked-in browser smoke tests; a slice is not done until its smoke test passes. Phase 4 — spawn three competitive review subagents in parallel to audit the resulting commits for bugs, regressions, security, architecture, missing tests, and drive-by edits to unrelated files. Use this whenever the user wants to add, plan, scope, design, spec out, slice, ship, or review a feature — especially on an existing project where the goal is to add new capability without altering what already works. Trigger phrases include "add a feature", "plan a feature", "I want to add", "new feature for our app", "help me scope", "spec out", "PRD", "before I implement", "vertical slices", "slice this plan", "rewrite as slices", "without breaking existing code", "without touching the rest", "minimal blast radius", "review this commit", "audit my code", "second opinion", "is this safe to ship", and basically any feature work on an existing codebase. Use this even when the user dumps a long messy paragraph of requirements — that is exactly when this workflow is most valuable.
---

# Feature slice

A 4-phase workflow for shipping features on brownfield production codebases without breaking things for real users. Adapted from a workflow that has shipped features across multiple production SaaS products with paying customers.

## Why this exists

Most AI coding advice assumes greenfield. On a greenfield toy you can break the app, drop the database, ship unsecured APIs, and fix it later. On an app with paying customers you cannot. AI is happy to produce grand plans and tons of code — your job is to keep it disciplined.

The trick is not a fancy harness or a specific model. It is a small set of prompts and guardrails, applied in order.

## The core principle: add, don't disturb

When a developer adds a feature to an existing codebase, their instinct is correct: **touch as little of the surrounding code as possible.** Every line of unrelated code you modify is a chance to break something that already works. Every refactor you smuggle into a feature PR is a regression vector hiding inside a "feature."

This skill enforces that instinct. Each vertical slice should be **additive by default** — new files, new modules, new routes, new components — and only modify existing code when there is no other way. When modification is unavoidable, the change should be **surgical**: the smallest diff that makes the feature work, with the rest of the file left exactly as it was.

This applies at every phase: Phase 1 asks "what can stay untouched?", Phase 2 prefers new files over edits to existing ones, Phase 4 flags any drive-by edits to unrelated code as a serious issue. The user did not ask you to clean up the codebase. They asked you to add a feature.

## Pick the right phase to enter

Before doing anything, decide where the user is in the workflow:

- **No plan yet, just a brain-dump or feature request** → start at Phase 1 (Clarify).
- **Has a plan or PRD organized by layer** (models, then DB, then API, then UI) → start at Phase 2 (Slice).
- **Has a sliced plan, ready to build** → Phase 3 (Implement one slice).
- **Just committed a slice or feature, wants a check before merging** → Phase 4 (Review).

If the user is starting from scratch, run Phases 1 → 2 → 3 → 4 in order. Each slice in Phase 3 should be followed by Phase 4 before the next slice begins.

---

## Phase 1 — Clarify relentlessly

**Goal:** turn the messy brain-dump into a small, explicit, agreed plan. No code, no multi-file PRD.

### Investigate before asking

If a question can be answered by inspecting the codebase, docs, config, tests, or existing conventions, **investigate those sources first.** The user's time is more valuable than yours.

Things worth checking before asking anything:
- Existing models and tables in the area being modified
- Testing conventions (framework, layout, mocking style)
- Auth / permissions model
- Migration / deployment approach
- Similar features already implemented in the repo
- **Extension points** — where the codebase already invites new code (plugin registries, route tables, hook points, event subscribers, feature flags). If the codebase has a clean place to add, use it instead of cutting into existing files.

### Walk the dimensions

Most features die in production not from the obvious things, but from a dimension nobody thought about — the new endpoint with no rate limit, the migration with no rollback path, the UI that breaks for screen readers, the feature that ships but nobody can tell whether anyone uses it because no analytics event was added.

Before opening the AskUserQuestion interview, walk the dimensions below. For each item, the answer must be one of:

- **In scope** — the feature needs this; it goes in the plan
- **N/A — <one-line reason>** — the feature does not touch this dimension
- **Constraint** — already determined by the brain-dump or the codebase

Items marked "In scope" become the targets of your AskUserQuestion interview. **N/A is acceptable; silence is not** — every dimension below must have an explicit answer in the final plan.

**Code & data**
- Schema migrations — forward *and* rollback path
- Data backfills / dual-writes for existing rows
- Backwards compatibility — API version skew, mobile clients on old versions, deserialization of pre-existing records
- Background jobs, queues, scheduled work, retry + dead-letter handling

**Surfaces**
- Frontend — new screens vs. edits to existing screens
- Mobile / responsive breakpoints
- Accessibility (a11y) — keyboard nav, screen readers, color contrast
- Internationalization (i18n / l10n) — copy, dates, currency, RTL
- APIs — new endpoints vs. breaking changes to existing endpoints
- Outbound comms — email, push notifications, webhooks, in-app messages

**Safety**
- Auth & permissions — who can do this, who explicitly cannot
- Input validation & injection surface (SQL, command, prompt, XSS)
- Privacy — PII handled, retention window, GDPR / CCPA implications, audit logs
- Rate limiting — per-user, per-tenant, per-endpoint

**Operations**
- Feature flag — for dark launch, % rollout, kill switch (default: yes for any non-trivial feature on a production app)
- Observability — metrics that prove the feature *works as intended*, not just *doesn't crash*
- Error tracking — what gets caught, what alerts wake someone up at 2am
- Performance — latency budget, bundle size impact, query cost, N+1 risk
- Structured logging — fields the new code emits so it can be debugged in production

**Business**
- Cost / billing impact — usage limits, tier changes, unit economics, third-party API spend
- Analytics — event instrumentation plan that answers "did anyone actually use this?"
- Documentation — user-facing docs, internal runbook, API docs, changelog entry
- Support enablement — what support says when a user asks about this feature
- SEO / search index — if the feature creates public or searchable content

### Interview with AskUserQuestion

Use the **AskUserQuestion** tool. One question at a time. Each question includes a recommended answer.

Resolve the design tree systematically:
1. Identify the next unresolved decision.
2. Resolve prerequisite decisions before dependent ones — never ask about UI before the data model is decided, never ask about caching before the query shape is decided.
3. Explore meaningful branches, edge cases, constraints, defaults, and failure modes.
4. Avoid assumptions when the answer could materially affect the result.

10–30 questions before the plan settles is typical. Do not cut the interview short to seem efficient — incomplete plans cost more than long interviews.

### Push back on scope

This is the single most valuable thing this phase does. Whenever a scope question comes up, **default to the minimum version that solves the stated problem.** Frame each scope question with the recommended answer being to defer:

> "Should we include X in v1, or ship the minimum and add X if users actually ask for it? (Recommended: ship minimum.)"

If the user keeps adding scope, call it out:

> "This is now noticeably bigger than the original request. Should we cut Y to keep v1 shippable, or accept the larger scope?"

Saying no early is the point. You are protecting the user from a plan they will regret.

### Output

When the plan is explicit, shared, and sufficiently complete, write it to a file (e.g., `plan.md`) so later phases can `@`-mention it. Structure:

- **Goal** — one sentence
- **In scope (v1)** — bullets
- **Out of scope (later)** — bullets, with the reason each was deferred
- **Key decisions** — each question and the agreed answer
- **Dimensions** — one line per dimension from the checklist above (`In scope` items get a one-line plan, `N/A` items keep their reason)
- **Files / modules affected** — paths
- **Open risks** — what could still bite

Then state the **next concrete action** in one sentence — not "implement the feature", something like "write the migration for the `events` table."

### Done when

- A new engineer could read the summary and start work without asking you anything.
- Every decision that could materially affect the result has an explicit answer.
- The scope fits in one shippable v1.

Move to Phase 2.

---

## Phase 2 — Rewrite into vertical slices

**Goal:** convert the plan from horizontal (models → DB → API → UI → test) into vertical (one feature end-to-end, then the next).

Horizontal plans only integrate at the very end, and that is exactly when integration breaks. Vertical slices pay down integration risk continuously.

### Rewrite rules

Group by **feature, workflow, or entity** — not by layer.

Each slice must be end-to-end:
- model
- DB (migration if needed)
- API / backend
- UI
- unit tests
- browser smoke test
- integration into the running app

Each slice is self-contained, tested, and shipped before the next starts. If a slice depends on something not yet built, it is in the wrong order — move it later.

### Prefer additive over destructive

For each slice, **list new files vs. modified files**. The ratio should lean heavily toward new. If a slice modifies more existing files than it creates, stop and ask whether the change can be restructured as:

- A new file that the existing code calls into (via a registry, hook, or feature flag)
- A new sibling module rather than an edit to an existing one
- An extension to a config / route table rather than a rewrite of the consumer

When an existing file *must* be edited, the diff should be **surgical**: add the necessary call site or wire-up, do not "while I'm here" rename variables, reformat, refactor, or reorganize. Drive-by changes hide regressions inside feature work and make rollback impossible.

In the slice output, explicitly list:
- **New files:** paths
- **Modified files:** paths, and one line per file explaining *why* the edit is unavoidable

### Smoke test per slice

For every slice, add a **browser smoke test** (Playwright, agent-browser, or whatever the repo already uses):
- Run the app
- Click through the UI for this slice's flow
- Enter sample data
- Trigger the main action
- Confirm both success and error states
- Verify the backend result if relevant

A slice is **not done** until its smoke test passes.

Unit tests as normal. Use full E2E sparingly — if every slice carries an E2E, most iterations will burn on test runs. Smoke + unit is the right balance for almost everything.

### Output format

Be aggressively concise. Compact phrasing over readability. No caveats, no nice-to-haves, no explanations of what each layer means — the reader is an engineer (or an agent) about to execute.

Use this exact template:

```
## Slice 1 — <short name>
**Ship:** <one-sentence user-visible outcome>
**New files:** <paths>
**Modified files:** <path — why edit is unavoidable>
**Model/DB:** <change>
**API:** <change>
**UI:** <change>
**Unit tests:** <list>
**Smoke test:** <click-through to verify in browser>
**Done when:** smoke test passes, slice is committed, modified-files list matches the diff.

## Slice 2 — <short name>
...
```

Order slices so each one is shippable on its own and each one builds on the previous. The first slice should produce *something a user can see*, even if narrow.

Move to Phase 3.

---

## Phase 3 — Implement one slice at a time

**Goal:** ship one slice end-to-end with the smoke test green before starting the next.

Do not start slice 2 before slice 1's smoke test passes and slice 1 is committed. The whole point of vertical slicing is wasted if you batch them.

After each slice ships:
1. Commit it (clean, focused commit, referencing the slice number or name).
2. Run Phase 4 on that commit.
3. Apply any critical / high issues from the review before moving on.
4. Then start the next slice.

---

## Phase 4 — Adversarial review (3 competing subagents)

**Goal:** catch what the implementer missed. A single review pass — even by a smart model — misses things because reviewer and implementer share blind spots. Three independent reviewers framed competitively do not.

### Step 1 — Identify what to review

Confirm with the user (or infer if obvious):
- Which commit(s) to review. Default: the most recent commit on the current branch.
- Which plan to audit against. Default: the most recent plan or PRD file the user references. If neither is clear, run `git log --oneline -10` and ask.

### Step 2 — Spawn three reviewers in parallel

Use the **Agent** tool with `subagent_type: general-purpose`. Send all three Agent calls in a **single message with three tool_use blocks** so they run concurrently. Do not run them sequentially — you lose the independence (a sequential reviewer can be primed by reactions to the previous one).

Give each subagent this prompt verbatim, substituting the commit and plan references:

> Independently audit commit `<commit-ref>` implementing @`<plan-path>`.
>
> Look for serious coding issues and plan misalignments:
> - bugs
> - broken edge cases
> - regressions in nearby code
> - security risks
> - bad architecture
> - missing tests
> - **drive-by edits**: changes to files unrelated to the feature (renames, reformats, refactors, "while I'm here" cleanups). These hide regressions inside feature work — flag any modified file that is not strictly required by the plan.
> - **missing dimensions**: things that should exist for a feature of this kind but don't — e.g., a new endpoint with no rate limit, a migration with no rollback, a UI change with no a11y consideration, a feature with no analytics event, no feature flag on a non-trivial change. Cross-reference the Phase 1 dimensions checklist; flag any `In scope` dimension that the diff failed to actually address.
>
> Incentive: the agent finding the most valid serious issues gets 5 points.
>
> Return your findings as a numbered list. For each issue: `file:line`, one-sentence description, severity (critical / high / medium / low), and a one-sentence fix suggestion.

Do not explain to the subagents what the points are for. The competitive framing is what does the work — explaining it dilutes the effect.

### Step 3 — Dedupe and prioritize

When all three return:
1. Merge their findings into one list.
2. Collapse duplicates — same `file:line` plus same root cause is one entry. Note how many reviewers caught it (useful confidence signal).
3. Sort by severity: critical → high → medium → low.
4. Drop pure style nits unless the user asked for those.

### Step 4 — Present

```
## Issues found (N total, deduped)

### Critical
1. [path/to/file.ts:42] <issue> — fix: <one-sentence fix>. (found by 2/3 reviewers)
2. ...

### High
...

### Medium
...
```

End with a one-line note on next steps for any critical or high issues — usually "fix before merging" for criticals, "fix or explicitly accept" for highs.

### Model choice

Reviews are most useful when run with a **different model family** than the one that wrote the code. Same-model review catches noticeably less, because reviewer and implementer share training-induced blind spots.

If the user is clearly using the same model for both implementation and review, mention this once — they may want to swap the reviewer model (e.g., implement with Claude, review with GPT, or vice versa) for the next round.

### Confidence signals

- Caught by **2+ reviewers independently** → almost always real, treat as confirmed.
- Caught by **1 reviewer** → worth a look but may be a false positive, read carefully.
- All three return short lists (under 3 items each) → code is probably solid.
- All three return long overlapping lists → slow down and fix before merging.

---

## The whole workflow at a glance

1. **Clarify** — lock scope before any code (10–30 questions via AskUserQuestion).
2. **Slice** — rewrite horizontal plan into vertical end-to-end slices with smoke tests baked in.
3. **Implement** — ship one slice at a time, smoke test green before the next starts.
4. **Review** — three competing subagents audit each commit before merge.

That is it. No complex setup, no special tool, no specific model required. Any SOTA model works.
