# agentic-harness.claude

A Claude Code plugin integrating [agentic-harness.core](https://github.com/Jitsusama/agentic-harness.core):
the same workflow domains [agentic-harness.pi](https://github.com/Jitsusama/agentic-harness.pi)
drives through pi's extension API, here through Claude Code's skills and
hooks instead. Both adapters call the same pi-agnostic core; neither owns
the domain logic.

## What's here

- `skills/tdd/` — teaches Claude Code test-driven development and how to
  drive core's TDD loop through the `agentic-harness-core` CLI (installed
  separately; see below).

Guardians and interceptors (commit review, git/GitHub CLI checks, ...) will
land here as `PreToolUse` hooks in a later phase — Claude Code hooks can
allow/deny/ask but can't rewrite a tool call the way pi's guardians can, so
each of those needs its own adapter decision, not a mechanical port.

## Requirements

The `agentic-harness-core` CLI must be on `PATH`. It's packaged separately
(not bundled in this plugin) since it's shared with the pi adapter.

## Development

Load this plugin directly during development:

```sh
claude --plugin-dir .
```

Then exercise `/agentic-harness:tdd` or let Claude invoke it automatically
when TDD is in scope.
