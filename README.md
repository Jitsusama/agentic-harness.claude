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
- `hooks/hooks.json` — a `PreToolUse` hook on `Bash` calling
  `agentic-harness-core hook pre-bash`, which denies commands that violate
  pi's git-cli/github-cli conventions (`git commit --amend`, an unquoted
  commit heredoc, a `gh pr create` with an inline `--body`, ...) with the
  same block reasons pi's interceptors give. A command this hook doesn't
  flag gets no explicit decision, so the normal permission flow (other
  hooks, the user's permission mode) still applies.

Guardians (commit review and the rest, which need pi's rewrite-as-inline-
edit capability that a hook can only approximate as deny-and-retry) are
still ahead, each needing its own adapter decision rather than a
mechanical port.

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
