---
name: notes
description: >
  Manage a flat, ID-addressable archive of markdown notes: create a
  note, find notes by type/tag/parent/date/text, show one, change
  its type or tags, set a parent for hierarchy, render the tree, or
  regenerate the index. Use in any repo that stores its notes this
  way -- the archive is wherever the CLI is invoked from, not a
  fixed location.
---

# Notes

A notes archive is a flat forest of self-contained directories, one
per note, living directly under the repo root -- not a
folder-per-topic tree. Modeled on [[quest]]'s own layout, and built
in the same library (`agentic-harness-core`): identity and hierarchy
live in front matter, not in where a file sits on disk.

```
NOTE-20130120-3MX6UQ/
  README.md
  attachment-1.jpg
  attachment-2.png
```

`NOTE-<created-date>-<suffix>` is the note's id: `YYYYMMDD` from its
creation date, then a six-character random suffix. Every note's
`README.md` carries front matter:

```yaml
---
id: NOTE-20130120-3MX6UQ
type: reference
title: "..."
created: 20130120T...Z
updated: 20140301T...Z
tags: ["..."]
parent: NOTE-...      # optional
source: "..."          # optional, wherever the note originally came from
---
```

`type` is one of a fixed, small vocabulary -- journal, reference,
faith, financial, gaming, travel, writing, inbox -- the note's kind,
not its subject (tags carry the subject). `inbox` is an honest
"uncertain, needs a human" bucket, not a dumping ground; use `find`
with `type: "inbox"` to work through it and `retype` to resolve it.
`parent` is optional and only set where a real hierarchy exists (a
tax-year note under an umbrella note, sub-articles under a guide) --
most notes have none.

## Where the archive lives

Unlike quest's `questsRoot` -- one shared, cross-repo location every
tool on the machine sees by default, because a quest tree is a
single campaign log -- a notes archive is a property of the repo
you're standing in. `notesRoot` defaults to the current working
directory. Point elsewhere with `--notes-root <path>` or the
`AGENTIC_HARNESS_NOTES_ROOT` env var.

## Commands

Every action is `agentic-harness-core notes <action>`, reading a
JSON params object from stdin (`{}` when the action needs none) and
writing a JSON result to stdout: `{"ok":true,"message":...,
"details":...}` or `{"ok":false,"guidance":...}`. Read `message` or
`guidance` first; `details` carries structured data for actions
that return one (a listing, a projection, a count).

```sh
echo '{}' | agentic-harness-core notes types
echo '{"title":"...","type":"reference","tags":"a,b","parent":"NOTE-..."}' | agentic-harness-core notes create
echo '{"type":"reference","tags":"linux","since":"20130101","until":"20140101","q":"text","limit":10}' | agentic-harness-core notes find
echo '{"id":"NOTE-..."}' | agentic-harness-core notes show
echo '{"id":"NOTE-...","type":"reference"}' | agentic-harness-core notes retype
echo '{"id":"NOTE-...","title":"New Title"}' | agentic-harness-core notes retitle
echo '{"id":"NOTE-...","add":"a,b","remove":"c"}' | agentic-harness-core notes tag
echo '{"id":"NOTE-...","parent":"NOTE-...  |  none"}' | agentic-harness-core notes reparent
echo '{"id":"NOTE-..."}' | agentic-harness-core notes tree
echo '{}' | agentic-harness-core notes reindex
```

`types` lists the vocabulary with live counts. `create` mints a
fresh id and scaffolds `README.md`; pass `created` (a `created`-style
timestamp or bare `YYYYMMDD`) to backdate it, the way an import
would. `find` filters by any combination of `type`, `tags` (a single
tag), `parent`, a `created` date range (`since`/`until`), and a
case-insensitive text search (`q`) over title + body; it's the
normal way to navigate an archive -- prefer it over `ls`/browsing,
since location carries no meaning. `show`'s `id` accepts an
unambiguous prefix, not just the full id -- so does every other verb
that takes one. `retype`/`retitle`/`tag`/`reparent` mutate front
matter in place; `retitle` also keeps the body's `# <title>` heading
in sync, since it's an echo of the same field, not independent
content. `reparent` refuses self-parenting and refuses a reparent
that would form a cycle, the same guard quest's own reparent uses.
`tree` renders the parent/child forest (or one subtree, given an
id). `reindex` regenerates the root `INDEX.md` -- a static,
human-browsable snapshot; it is not the source of truth and can go
stale, so re-run it after bulk changes rather than trusting it
blindly.

## What's not here

No stage machine (quest's think/draft/build/conclude) -- these are
archived notes, not active work, so there's nothing to move through
stages. No cross-process lock (quest needs one because two pi
sessions can legitimately attach to the same quest concurrently; a
notes archive has no equivalent multi-writer scenario). No undo --
`retype`/`tag`/`reparent`/`retitle` overwrite in place; rely on git
for history.

## A note on formatting drift

The front-matter serializer uses the real `yaml` library, the same
choice quest's own frontmatter module makes, rather than hand-rolled
parsing -- an earlier prototype of this tool hand-rolled its own
escaping and accumulated corrupted titles across repeated edits
before that was caught. One consequence: a note imported by an older
tool with a different formatting convention (e.g. flow-style
`tags: ["a", "b"]` instead of block-style) will have its formatting
rewritten to this library's natural output the first time any verb
here touches it. That's expected, not a bug -- the tool doesn't fight
the library for cosmetic byte-parity with whatever produced the file
before, the same way quest doesn't.
