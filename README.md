# feature-slice-skill

A [Claude Code](https://claude.com/claude-code) skill for **adding features to existing codebases one vertical slice at a time, without disturbing the surrounding code.**

When a developer adds a feature to an existing project, their instinct is to touch as little of the surrounding code as possible — every line of unrelated code you modify is a chance to break something that already works. This skill enforces that instinct: each slice is **additive by default** (new files, new modules), and when an existing file *must* be edited, the diff is **surgical** — no drive-by renames, no "while I'm here" refactors. Works on both greenfield and brownfield, but it earns its keep on brownfield: code with real users where you can't just "fix it later."

## The 4-phase workflow

1. **Clarify** — Run a relentless clarification interview using the `AskUserQuestion` tool. One question at a time, each with a recommended answer. The skill pushes back on scope creep and produces an explicit, agreed plan before any code is written.
2. **Slice** — Rewrite the plan into vertical end-to-end slices (model + DB + API + UI + tests + integration), not horizontal layers. Horizontal plans only integrate at the very end, which is exactly when integration breaks.
3. **Implement** — Ship one slice at a time. A slice is not done until its **browser smoke test** passes. Then commit and move on.
4. **Review** — Spawn 3 review subagents in parallel, framed as a competition. They independently audit the commit for bugs, regressions, security issues, bad architecture, and missing tests. Their findings get deduped and prioritized.

The skill is designed so each phase can be entered independently — drop the user at Phase 4 with "review this commit" and it just works, or start fresh at Phase 1 with a feature request.

## Installation

```sh
npx skills add klind/feature-slice-skill
```

Uses the [`skills`](https://github.com/vercel-labs/skills) CLI — works with Claude Code, Cursor, Codex, OpenCode, and others. It will prompt for which agent and whether to install globally (`~/.claude/skills/feature-slice-skill/`) or per-project (`./.claude/skills/feature-slice-skill/`).

### Non-interactive (CI / scripts)

```sh
npx skills add klind/feature-slice-skill -g -a claude-code -y
```

### Manual install (no npx)

```sh
git clone https://github.com/klind/feature-slice-skill.git /tmp/fs
cp -r /tmp/fs/feature-slice-skill ~/.claude/skills/feature-slice-skill
```

## Usage

The skill triggers automatically when you say things like:

- "I want to add [feature] to [our app]"
- "Plan a feature for…"
- "Help me scope this"
- "Slice this plan"
- "Rewrite as vertical slices"
- "Review this commit"
- "Second opinion on what I just shipped"

Or invoke it explicitly:

```
/feature-slice-skill
```

Then dump everything you know about what you want to build — technical details, business goals, in-scope, out-of-scope, links to tickets, user complaints, whatever you have. The messier the better. The skill takes it from there.

## Why this works

- **Add, don't disturb.** Every slice lists new files vs. modified files explicitly. When the modified-files list is longer than the new-files list, the slice is wrong and gets restructured. Drive-by edits ("while I'm here" renames, reformats, refactors) are treated as a serious issue at review time, because they hide regressions inside feature work and make rollback impossible.
- **Locking scope before any code** prevents the "AI produces grand plan, you regret it later" failure mode. Most of the value is in Phase 1 saying no to things.
- **Vertical slices** mean every slice is integration-tested before the next starts. You never reach the end of a 5-day implementation and discover the layers don't fit together.
- **Browser smoke tests per slice** give a tight feedback loop on a real user flow — type checks and unit tests verify code correctness, not feature correctness.
- **Three competing reviewers** catch what one reviewer (sharing blind spots with the implementer) misses. The competitive framing is doing real work — LLMs respond to it without needing to be told why.

## Credit

The workflow is adapted from a blog post on shipping AI-coded features to production SaaS apps with paying customers. The four prompts at the heart of this skill (relentless clarification, vertical slice rewrite, smoke-tests-per-slice, three competing reviewers) are taken from that post almost verbatim — the value of this skill is having them codified so Claude Code picks them up automatically.

## License

MIT
