---
name: quest
description: >
  Track multi-session work as a quest: a durable, on-disk campaign
  log with documents (plans, research, briefs, reports) that move
  through a think/draft/build/conclude stage machine. Use to create
  or resume a quest, drive a document through its stages, or find,
  reparent and reorder quests. Shared with the pi adapter on the
  same machine: the same quest tree, not a separate one.
---

# Quest

A quest is a durable campaign for one piece of work, living as a
directory of plain markdown under a shared quest tree — not a
session, not a todo list. Documents (plans, research, briefs,
reports) live under it and move through a stage machine: think,
draft, build, then conclude or retire.

**This quest tree is shared with the pi adapter on the same
machine.** `agentic-harness-core` resolves the quests root to the
exact path pi's own quest-workflow extension writes to by default,
so a quest created here is loadable from pi and back — one campaign
log, not two. Override with `--quests-root <path>` or the
`AGENTIC_HARNESS_QUESTS_ROOT` env var if that's not what's wanted.

## Commands

Every action is `agentic-harness-core quest <action>`, reading a
JSON params object from stdin (`{}` when the action needs none) and
writing a JSON result to stdout: `{"ok":true,"message":...,
"details":...}` or `{"ok":false,"guidance":...}`. Read `message` or
`guidance` first; `details` carries structured data for actions
that return one (a listing, a projection, an id).

Run it from the project root so its state file
(`.agentic-harness/quest-state.json`, created on first use) tracks
the loaded quest and focused document across calls the same way a
shell tracks its cwd. Add `.agentic-harness/` to the project's
`.gitignore` if it isn't already ignored.

### Lifecycle

```sh
echo '{"title":"Ship the thing","kind":"quest"}' | agentic-harness-core quest create
echo '{"id":"QEST-20260819-AB12CD"}' | agentic-harness-core quest load
echo '{}' | agentic-harness-core quest show
echo '{}' | agentic-harness-core quest list
echo '{}' | agentic-harness-core quest unload
```

`create` takes `title` (required unless `url` resolves one),
`kind` (`quest`, `subquest` or `sidequest`; defaults to
`sidequest`), `parent`, `priority` and `note`, and loads the new
quest. `load` resolves by `id`. `show` with no `id` projects the
loaded quest; with an `id` it projects any quest read-only without
changing what's loaded. `list` takes optional `kind`/`status`/
`priority`/`parent`/`limit`/`offset` filters.

### The Document Stage Machine

```sh
echo '{"note":"Investigating the auth flow","kind":"plan"}' | agentic-harness-core quest think
echo '{"title":"Auth rework plan"}' | agentic-harness-core quest draft
echo '{}' | agentic-harness-core quest build
echo '{}' | agentic-harness-core quest conclude
echo '{"reason":"Superseded by a simpler approach"}' | agentic-harness-core quest retire
echo '{}' | agentic-harness-core quest reopen
```

`think` opens a loop with no document id yet; `draft` (with a
`title`) mints one. `conclude`/`retire` with no `id` and no
`scope` act on the focused document if one is focused, otherwise
on the whole loaded quest (pruning any trees the quest scaffolded).
Pass `id` to target a specific document or quest instead, or
`scope: "document"` / `scope: "quest"` to be explicit.

### Focus

```sh
echo '{"id":"PLAN-20260819-AB12CD"}' | agentic-harness-core quest focus
echo '{}' | agentic-harness-core quest unfocus
```

### Reorder and Priority

```sh
echo '{}' | agentic-harness-core quest top
echo '{}' | agentic-harness-core quest bottom
echo '{}' | agentic-harness-core quest bump
echo '{}' | agentic-harness-core quest sink
echo '{"target":"QEST-..."}' | agentic-harness-core quest before
echo '{"target":"QEST-..."}' | agentic-harness-core quest after
echo '{}' | agentic-harness-core quest promote
echo '{}' | agentic-harness-core quest demote
echo '{}' | agentic-harness-core quest drive
echo '{}' | agentic-harness-core quest park
echo '{}' | agentic-harness-core quest defer
```

`top`/`bottom`/`bump`/`sink`/`before`/`after`/`renumber` move a
quest within its sibling rank (same parent, same priority bucket).
`promote`/`demote` shift one priority bucket; `drive`/`park`/
`defer` jump straight to `driving`/`bench`/`someday`.

### Structural

```sh
echo '{"id":"QEST-...","parent":"QEST-..."}' | agentic-harness-core quest reparent
echo '{"id":"QEST-...","parent":null}' | agentic-harness-core quest reparent
echo '{}' | agentic-harness-core quest undo
```

`reparent` takes a comma-separated `id` list for a batch move, and
a `dryRun: true` preview before committing. `undo` reverses the
most recent structural op (reparent, bulk conclude/retire) — one
level, not a full history.

### Alias

```sh
echo '{"ref":"github:owner/repo#42"}' | agentic-harness-core quest alias-add
echo '{"ref":"github:owner/repo#42"}' | agentic-harness-core quest alias-remove
```

An alias binds a quest to an external reference (a GitHub issue, a
URL); `create` with a `url` that resolves to a registered ref type
seeds one automatically.

### Trees

```sh
echo '{"cwd":"/path/to/repo"}' | agentic-harness-core quest tree-add
echo '{"cwd":"/path/to/repo","force":true}' | agentic-harness-core quest tree-adopt
echo '{}' | agentic-harness-core quest tree-list
echo '{}' | agentic-harness-core quest tree-prune
```

`tree-add` scaffolds a fresh git worktree for the loaded quest;
`tree-adopt` registers an existing checkout instead, without
creating anything (needs `force: true` for a repo's main working
tree or one another quest already tracks). A scaffolded tree is
auto-pruned when its quest concludes or retires; an adopted one
never is.

### Queries

```sh
echo '{"query":"auth rework"}' | agentic-harness-core quest find
echo '{"role":"reviewer"}' | agentic-harness-core quest who
echo '{}' | agentic-harness-core quest links
echo '{"id":"QEST-..."}' | agentic-harness-core quest locate
echo '{"id":"QEST-..."}' | agentic-harness-core quest ancestors
echo '{}' | agentic-harness-core quest tree
echo '{"id":"QEST-..."}' | agentic-harness-core quest expand
```

`find` filters by `query`/`kind`/`status`/`priority`/`parent`/
`since`/`until`/`field` (`started`, `updated`, `due` or `eta`).
`who` searches cast bullets across every quest. `links` projects
the loaded quest's outgoing and incoming references. `locate`
resolves an id, alias or document id to its owning quest. `tree`
renders the whole forest; `expand` renders one subtree.

### Config

```sh
echo '{}' | agentic-harness-core quest config
```

Reports the resolved quests root, so you can confirm it lines up
with pi's before assuming a quest is missing rather than just not
loaded from this location.

## What's Not Here

Session tracking (`session-attach`/`session-detach`/`session-rename`,
automatic attach on `load`, per-session liveness in `show`/`list`),
terminal spawning (`spawn-tab`/`spawn-pane`/`spawn-window`) and the
lost-session recovery verbs (`workspace`/`recent`/`restore`) are all
keyed to pi's own session JSONL log and terminal-reopen mechanism,
which nothing here has an equivalent for. A quest created or driven
from here is fully visible to pi — these are pi-side conveniences on
top of the same shared data, not gaps in the data itself.

Cast subjects in `show` are not resolved to identities (pi's
`lib/people` identity registry isn't ported here yet); you'll see
the raw `@handle` or name as written.
