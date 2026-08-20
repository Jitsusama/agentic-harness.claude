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
  `agentic-harness-core hook pre-bash`, which:
  - denies commands that violate pi's git-cli/github-cli conventions
    (`git commit --amend`, an unquoted commit heredoc, a `gh pr create`
    with an inline `--body`, ...), with the same block reasons pi's
    interceptors give.
  - installs a commit-attribution git hook (and refreshes its marker
    file) in whatever repo the command's effective cwd resolves to, on
    every bash call, so coverage follows the agent between repos the
    same way pi's own hook installation does.
  - denies an unattributed `gh pr`/`gh issue create`/`edit` with the
    exact corrected (attributed) command for the agent to retry with,
    since a hook can't rewrite the call in place the way pi's
    attribution-interceptor can.
  - asks before a destructive git command (`git push --force`,
    `git reset --hard`, `git rebase`, `git clean -f`, `git branch -D`,
    ...), naming the risk and whether it's recoverable via reflog.
    history-guardian's own review never rewrites, so `ask` is a
    complete adapter for it: Claude Code's native permission prompt
    stands in for pi's own allow/block confirmation exactly, with no
    deny-and-retry approximation needed. A hard-rule deny above still
    wins over an ask when a command trips both.

  A command this hook doesn't flag gets no explicit decision, so the
  normal permission flow (other hooks, the user's permission mode)
  still applies.

Commit attribution here is marker-file-gated, not env-var-gated like
pi's: a `PreToolUse` hook is a separate process that exits before the
tool call it precedes runs, with no way to inject an env var forward
into it. The trade-off is that this can't distinguish an agent-driven
commit from a human one typed in the same repo while the plugin is
active, unlike pi's per-call env var. Given the domain's own transparency-
first ethos, that's an accepted trade, not an oversight.

Other guardians (commit review, PR/issue review, which need pi's
rewrite-as-inline-edit capability that a hook can only approximate as
deny-and-retry) are still ahead, each needing its own adapter decision
rather than a mechanical port.

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
