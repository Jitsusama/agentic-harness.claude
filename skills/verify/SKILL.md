---
name: verify
description: >
  Run the project's verification check command (lint, typecheck,
  test) and report whether the code still builds and passes. Use
  when asked whether something works, still builds, or is green,
  rather than guessing or re-reading code to check by eye.
---

# Verify

Run the project's resolved check command through the
`agentic-harness-core` CLI instead of guessing whether the code
still builds:

```sh
agentic-harness-core verify run
```

Run it from the project root. It walks up from the current
directory to the nearest `package.json`, resolves a check command
by precedence (an explicit `verify` script, otherwise whichever of
`lint`, `typecheck` and `test` scripts exist, joined so all run),
runs it, and reports the result as JSON on stdout:

```json
{"ok":true,"command":"npm run lint && npm run test","output":"Passed: npm run lint && npm run test\n\n...tail of output..."}
```

or, on failure:

```json
{"ok":false,"command":"npm run lint","output":"Failed (exit 1): npm run lint\n\n...full output..."}
```

The exit code mirrors `ok` (0 on pass, 1 on fail), and `output` is
already a human-readable summary: a short tail on a pass, the full
captured output on a failure. Report `output` back to the user
directly rather than re-summarizing it. When `ok` is `false`,
there's no need to guess further; act on what the output names.

If `output` says no verification command was found, there's no
lint, typecheck, test or verify script in the resolved
`package.json` — say so rather than inventing a command to run.

This is an on-request check only: nothing runs automatically after
an edit. Run it when the user asks whether the code works, still
builds, or is green, or before telling them it does.
