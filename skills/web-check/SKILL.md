---
name: web-check
description: >
  Judge a live web page for the four things static checks cannot
  see: whether it honours prefers-reduced-motion, orphans and
  runts in real line wrapping, whether the server render survived
  hydration, and web vitals rated as medians over several loads.
  Use when asked to QA a page, review a preview URL, check
  reduced motion or hydration, or measure page performance
  honestly.
---

# Web Check

Run one judgment against one URL through the
`agentic-harness-core` CLI, which opens a headless Chrome, loads
the page, runs the check and closes the browser:

```sh
echo '{"kind":"motion","url":"https://example.com"}' | \
  agentic-harness-core web check
```

The four kinds, and when to reach for each:

- `motion`: emulates `prefers-reduced-motion: reduce`, reloads,
  and reports what kept moving. Video with no way to stop it and
  animation past the five second line fail WCAG 2.2.2;
  scroll-driven motion fails 2.3.3; a brief fade is left alone.
  This is the one accessibility setting a page can ignore
  without failing a single axe rule, so no other check sees it.
- `typography`: reads real line boxes word by word and reports
  orphans (one word alone on a last line) and runts (a stub of a
  last line), worst in headings. Taste, not conformance: it
  never says more than WARN.
- `hydration`: fetches the server's HTML from inside the page,
  parses it without running a script, and diffs it against the
  hydrated document, reading the console for the framework's own
  complaints. Vanished server content fails; content that only
  exists after hydration warns, because a client-only widget is
  legitimate and the check cannot tell it from a bug.
- `perf`: web vitals against their published thresholds, rated
  as the median over several loads (default 3, `"samples": N` to
  change) with the spread beside each number. A rating that
  differed between loads is called out, and an unstable pass
  reads WARN, because a single headless load drifts.

The answer is JSON on stdout:

```json
{"ok":true,"kind":"motion","url":"...","report":"PASS  ..."}
```

`report` opens with PASS, WARN or FAIL, where WARN means
undecided rather than nearly fine: a check that could not judge
says so there instead of passing quietly. Relay `report` to the
user as it stands rather than re-summarizing it; the wording
carries the caveats (what was measured, what was left out, what
needs a person).

On a refusal (`ok: false`), `error` names what was wrong: a URL
that would not load, a page that answered an error status, or an
unknown kind. Say what it says rather than retrying blind.

Findings here are symptoms measured from a real render. When you
go on to claim *why* the page behaves this way, quote the
responsible source to a file and line, or phrase the mechanism
as a question; the report never claims a cause and neither
should a relay of it.
