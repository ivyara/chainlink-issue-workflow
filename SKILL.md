---
name: chainlink-issue-workflow
description: Use whenever a repo uses Chainlink, `.chainlink/`, `chainlink` issue tracking, local sessions, handoffs, multi-agent locks, or instructions to create issues before work. Guides agents through Chainlink during implementation, debugging, reviews, context handoffs, and coordination. Also use for "continue work", "pick next issue", "track this", "handoff", or "use the local issue tracker."
---

# Chainlink Issue Workflow

Chainlink is a local-first issue tracker for AI-assisted development. Use it as durable task state when a project has `.chainlink/` or instructions mention Chainlink. Command surfaces vary by version; discover capabilities before advanced commands.

## Start

Before code edits:

1. Run `chainlink --help`, then `chainlink session status`.
2. If no session is active: `chainlink session start` and read handoff notes.
3. If no issue is active: use `chainlink next` for a suggestion and a list of other ready tasks, then choose or create one and set it active.
4. If lock commands exist: check/claim the lock before editing.

Baseline flat commands: `chainlink quick|create|subissue|list|show|comment|ready|next|close`, plus `chainlink session start|status|work|action|last-handoff|end`.

Newer builds may expose `chainlink issue ...`, `timer ...`, `agent ...`, `locks ...`, typed comments, and typed relations. Use these only after `chainlink --help` or subcommand help confirms support.

## Command Selection

- If `chainlink issue` exists, grouped forms such as `chainlink issue quick` are allowed; otherwise use flat forms such as `chainlink quick`.
- If `locks`/`agent` are absent, coordinate with comments, dependencies, and the user; do not invent lock commands.
- If `chainlink comment --help` lacks `--kind`, prefix comments with `Plan:`, `Result:`, `Blocker:`, `Decision:`, `Handoff:`.
- If `chainlink relate --help` lacks a relation type option, use plain `chainlink relate <id1> <id2>`.
- If a command shape fails, check help once and switch to the supported equivalent.

## Issues

Create an issue before implementation, debugging, refactoring, tests, or review fixes.

```bash
chainlink quick "Fix login timeout on slow connections" -p high -l bug
chainlink create "Fix login timeout on slow connections" -p high -l bug --work  # if quick is absent
chainlink create "Add user authentication flow" -p high -l feature
chainlink subissue 1 "Add registration endpoint" -p high -l feature
chainlink subissue 1 "Add login endpoint" -p high -l feature --work
```

Use equivalent `chainlink issue ...` forms only when grouped commands are confirmed.

Titles should be changelog-ready: start with a verb (`Add`, `Fix`, `Update`, `Remove`, `Improve`), describe the user-visible outcome, and avoid junk like `WIP`, `fix bug`, or `auth.ts changes`.

Priorities: `critical` for outages/security/blockers; `high` for direct user requests/core functionality; `medium` for normal features/fixes/investigations; `low` for cleanup/optimization. Labels: `bug`/`fix`, `feature`/`enhancement`, `breaking`, `security`, `deprecated`, `removed`.

Split large work into subissues; work one at a time.

## Work Log

Keep durable context on the active issue:

```bash
chainlink session work <id>
chainlink comment <id> "Plan: update token refresh path, then add regression test"
chainlink comment <id> "Decision: reuse existing helper in src/auth/session.ts"
chainlink session action "Tracing token refresh failure through auth middleware"
```

If `--kind` is supported, typed comments are fine: `chainlink comment <id> "Update token refresh path" --kind plan`.

Use comments for decisions, blockers, test results, root cause, and handoff context. Use `session action` for short breadcrumbs during long tasks or before context compression.

## Dependencies And Coordination

Use explicit relationships for ordering and context:

```bash
chainlink block <blocked-id> <blocker-id>
chainlink ready
chainlink relate <id1> <id2>
chainlink related <id>
```

If typed relations are supported: `chainlink relate <assumption-id> <dependent-id> --type falsifies`.

For multi-agent work, only run `chainlink locks --help` or `chainlink agent --help` if those commands appear in top-level help. If available:

```bash
chainlink agent status
chainlink locks check <id>
chainlink locks claim <id> --branch <branch>
chainlink locks release <id>
```

Initialize an agent identity only when authorized by project instructions or the user. If locks are absent, add a `Claim: ...` comment and ask before touching work that appears owned. Never edit an issue locked by another agent; only `locks steal` when explicitly allowed for stale locks.

## Close And Handoff

Verify behavior, record the result, then close:

```bash
chainlink comment <id> "Result: npm test and npm run lint pass"
chainlink close <id>
```

Use `chainlink close <id> --no-changelog` only when help confirms support and the issue is internal tracking/scaffolding.

End sessions when stopping, near context limits, or leaving follow-up work:

```bash
chainlink session end --notes "Completed #12. Remaining: #13 needs browser verification; blocker is missing test account."
```

Handoff notes should include completed work, current state, tests run, remaining work, blockers, assumptions, and useful files.

## Scripting And Failure Handling

- Use JSON only when the command supports it: `chainlink --json show <id>`, `chainlink --json list -s open`, `chainlink --json session status`; grouped JSON/locks only after help confirms support.
- Use quiet mode for captured IDs: `chainlink -q create "Fix checkout validation" -p high -l bug`.
- If `chainlink` is missing, report that the project expects it and continue only if the user wants to proceed untracked.
- If `.chainlink/` is missing, run `chainlink init` only when the user or project instructions require it.
- If hooks block edits due to no active issue, create/select an issue and set it active. Do not bypass hooks or edit `.claude`/`.chainlink` safety files.

Fallbacks:

- `chainlink issue quick ...` -> `chainlink quick ...`
- `chainlink issue list|show|comment|close` -> `chainlink list|show|comment|close`
- `chainlink comment <id> "..." --kind result` -> `chainlink comment <id> "Result: ..."`
- `chainlink locks ...` unavailable -> issue comments/user coordination

## Templates

New work:

```bash
chainlink --help
chainlink session start
chainlink quick "Add <user-visible capability>" -p high -l feature
chainlink comment <id> "Plan: <short plan>"
# implement, test
chainlink comment <id> "Result: <commands and result>"
chainlink close <id>
chainlink session end --notes "Completed <id>. <next useful context>"
```

Continue work:

```bash
chainlink --help
chainlink session start
chainlink session last-handoff
chainlink next
chainlink show <id>
chainlink session work <id>
```

For investigations, use the new-work flow with a `bug` label plus `Reproduction:` and `Root cause:` comments. Use `--template investigation` only if `quick/create --help` confirms that template.
