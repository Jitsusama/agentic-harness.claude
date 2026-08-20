---
name: memory
description: >
  Durable, project-scoped facts that survive across sessions:
  decisions, where things live, rationale worth keeping. Use to
  remember something worth not re-deriving later, and check for
  remembered facts before re-deriving context you may have
  already recorded.
---

# Memory

Facts retained here persist across sessions and are shared with
the pi adapter on the same project: something retained while
working through pi is recallable here, and back.

Unlike pi, nothing rehydrates these into context automatically —
there is no standing mechanism here that injects text into every
turn. **Proactively call `memory recall` near the start of
non-trivial work on a project**, before re-deriving context you
may have already recorded. This is the one thing you have to
remember to do that pi does not.

## Commands

All four subcommands read a JSON request from stdin and write a
JSON response to stdout. Run from the project you want the facts
scoped to; scope defaults to the project at the current directory
when not given explicitly.

### Retain

Remember a durable fact. Use for decisions, where things live, and
rationale worth keeping across sessions — not for transient detail
that only matters for the current turn.

```sh
echo '{"text":"The API key lives in .env.local, not .env","tags":["config"]}' \
  | agentic-harness-core memory retain
```

`tags` and `source` are optional.

### Recall

Recall facts for the current scope, optionally filtered by a
keyword or tag substring.

```sh
echo '{"query":"api key"}' | agentic-harness-core memory recall
echo '{}' | agentic-harness-core memory recall
```

Returns an array of facts. Narrow with `query`, or lower `limit`
(default 20), when the result is too broad.

`query` is a literal substring match against a fact's text and
tags, not a fuzzy or per-word search: a fact edited to reword it
can stop matching a query that used to find it, even though the
fact itself is untouched (confirmed live: editing "the demo test
runner is..." to "...node --test (built-in runner)..." broke a
`"test runner"` query while a single-word `"vitest"` query kept
matching). An empty result means "no fact matches this exact
phrase," not "no fact exists" — try a shorter or different
substring, or `{}` with no query, before concluding nothing is
recorded.

### Reflect

Ask memory to synthesize a short answer over the facts it holds,
rather than listing raw facts.

```sh
echo '{"question":"where does the API key live"}' \
  | agentic-harness-core memory reflect
```

### Edit

Amend a fact's text or tags, or invalidate it when it is wrong or
superseded. An invalidated fact is never recalled again.

```sh
echo '{"id":1,"text":"corrected text"}' | agentic-harness-core memory edit
echo '{"id":1,"invalidate":true}' | agentic-harness-core memory edit
```

## Scope

Facts are scoped to the project they were retained in by default
(the directory `agentic-harness-core` runs from). A `global` scope
is always included in recall alongside the current one. To target
a specific scope explicitly, pass `"scope"` in the request body:

```json
{"text": "...", "scope": {"kind": "project", "path": "/abs/path"}}
{"text": "...", "scope": {"kind": "global"}}
```

Quest-scoped facts (`{"kind": "quest", "id": "..."}`) exist in the
schema for pi's quest-workflow, which has no equivalent here yet;
there's no reason to use that scope from this adapter today.
