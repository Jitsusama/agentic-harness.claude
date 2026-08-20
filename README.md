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
  - runs commit/PR/issue review: the same content gates pi's own
    guardians run (prose, section, title, redirect), denying outright
    on a violation exactly as pi does, then asking with the message
    or title/body once a command clears every gate. pi's own
    guardians already fall back to allowing outright in a context
    with no UI to show a panel in - the content gates are the actual
    enforcement there too, the panel only the confirmation on top -
    so asking here is not an approximation of anything pi does; it's
    genuinely more protection than pi's own headless fallback offers,
    since an ordinary Claude Code chat has a human present. None of
    the three rewrites either, so `ask` is complete for all of them
    the same way it is for history-guardian.

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

review-integration (the review-tool workflow, not the guardian) is
still ahead: its own gate is a richer interactive surface than a
single ask/deny decision covers, and needs its own adapter design
rather than the pattern the guardians above share.

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
