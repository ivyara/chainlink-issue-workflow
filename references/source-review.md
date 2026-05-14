# Chainlink Source Review

Source reviewed: `https://github.com/dollspace-gay/chainlink`, commit `5888134` from the shallow clone used during skill drafting.

## What Chainlink Is

Chainlink is a Rust CLI issue tracker designed for AI-assisted development. It stores data locally in SQLite under `.chainlink/issues.db`, supports session handoff, issue comments, priorities, labels, dependencies, subissues, milestones, time tracking, JSON output, and optional multi-agent locks.

The VS Code extension is present, but the skill should focus on the CLI because that is what coding agents can invoke reliably.

## Important Command Surface

Canonical commands are grouped under nouns:

- `chainlink init`
- `chainlink issue create|quick|subissue|list|search|show|update|close|close-all|reopen|delete`
- `chainlink issue comment|label|unlabel`
- `chainlink issue block|unblock|blocked|ready`
- `chainlink issue relate|unrelate|related|cascade|falsify`
- `chainlink issue next|tree|tested`
- `chainlink session start|end|status|work|last-handoff|action`
- `chainlink locks list|check|claim|release|steal`
- `chainlink agent init|status`
- `chainlink timer start|stop|show`
- `chainlink milestone ...`
- `chainlink archive ...`
- `chainlink export|import`

The CLI also retains hidden flat aliases such as `chainlink quick`, `chainlink create`, `chainlink list`, `chainlink comment`, and `chainlink close`. Project docs still show some flat forms, so the skill should mention both while preferring canonical forms.

## Agent Workflow Lessons

- Start or inspect a session before work. `session start` displays the prior handoff notes.
- Always set an active issue before code edits in projects using strict Chainlink hooks.
- Use `issue quick` for create + label + active work in one command.
- Use changelog-ready issue titles because closed issues can update `CHANGELOG.md`.
- Use comments for durable context that another agent can consume.
- Use `session action` for short breadcrumbs during long-running work or before context compression.
- Split large work into parent issues and subissues.
- Use dependencies and `issue ready` to avoid working on blocked items.
- Use locks for multi-agent work; do not edit issues locked by another agent.
- Use `--json` for structured reads and `-q` for scripts that need only an ID.
- End sessions with handoff notes that include completed work, current state, test results, and next steps.

## Hook Behavior To Teach Agents

Bundled Claude hooks demonstrate project expectations:

- `work-check.py` can block `Write`, `Edit`, and mutating `Bash` unless an active Chainlink issue exists.
- Git mutation commands can be permanently blocked by hook config.
- Editing hook infrastructure can be blocked.
- Lock conflicts can block or warn depending on tracking mode.
- `post-edit-check.py` detects stubs and reminds about tests.
- `prompt-guard.py` injects global, project, quality, rigor, and language-specific rules from `.chainlink/rules/`.

The skill should tell agents to comply with hooks, not bypass them.

## Design Choice For This Skill

The skill is intentionally command-oriented and workflow-oriented. It does not copy the whole Chainlink README because agents need a compact procedure:

1. Start/read session.
2. Create or select active issue.
3. Claim/check lock when relevant.
4. Comment durable discoveries.
5. Record breadcrumbs.
6. Test and close issue.
7. End session with handoff notes.

