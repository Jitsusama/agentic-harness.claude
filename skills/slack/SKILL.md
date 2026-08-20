---
name: slack
description: >
  Set up or check Slack access for this machine, via a browser
  session extraction that needs no Slack app or admin approval. Use
  when asked to connect Slack, check whether Slack is set up, or
  when a Slack-reading task needs credentials that aren't there yet.
---

# Slack Setup

Check and establish Slack access through the `agentic-harness-core`
CLI. Credentials are stored once per machine (under
`XDG_CONFIG_HOME`, not per-project), so most sessions only need the
status check.

## Check status

```sh
agentic-harness-core slack-auth status
```

Reports one of:

```json
{"authenticated":true,"team":"Acme","user":"joel"}
{"authenticated":false}
{"authenticated":false,"reason":"the stored session no longer works; run login again"}
```

Check this before telling the user Slack isn't set up, and before
any Slack-reading task that would otherwise fail on missing
credentials.

## Set up access

```sh
agentic-harness-core slack-auth login
```

This opens a real, visible Chrome window straight to Slack's sign-in
page. **Tell the user these steps before running it** (the command
also prints the same guidance to stderr the moment the window opens,
but say it in chat too - the window opens immediately and they
shouldn't have to wait on anything to know what to do):

1. Sign in with your email (or SSO) as usual.
2. If asked to pick a workspace, click into the one you want - the
   browser profile is fresh every run, so there's no existing
   session to skip this with.
3. If a screen offers "Open Slack" (desktop app) vs "use Slack in
   your browser", click "use Slack in your browser" - this tries
   that automatically, but don't wait on it if it looks slow.

The command blocks until it detects the session is live or five
minutes pass, so don't cancel it early just because it hasn't
returned yet.

On success:

```json
{"ok":true,"team":"Acme","user":"joel"}
```

On failure (Chrome not found, timed out waiting for login):

```json
{"ok":false,"error":"..."}
```

Report the `error` text back to the user directly rather than
guessing at the cause.

## What this does and doesn't cover

This is the browser-extraction path only - no Slack app, no client
ID or secret, no admin approval, works with any workspace the user
can log into normally. It is deliberately the *only* path wired up
here: it's genuinely zero-setup, unlike the OAuth-app alternative
(create a Slack app, get it approved, exchange a code), which is a
real option but not worth the extra steps until something actually
needs the narrower, app-scoped permissions it would grant instead of
a full user session.
