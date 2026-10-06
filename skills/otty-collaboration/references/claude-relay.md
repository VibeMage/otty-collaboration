# Relay from Codex to an existing Claude

Use only for user-authorized local collaboration. Rediscover the owner and target
with Otty pane/tab inventories and capture the target's context. Resolve its
session, PID, cwd and `/tmp/cc-socks/<pid>.sock` exactly as in SKILL.md; never
reuse a saved socket. Inspect process environments internally and print only the
matching pane/PID/cwd, not the full environment. Respect an awaiting human
approval and any native refusal; do not change permissions to make delivery work.

Prepare the complete handoff in a file, with the verified Codex owner pane/session
and tracker. Tell the recipient to reply to that owner, **not the ephemeral
relay**, using Otty Enter with delivery verification and a tracker copy.

The following CLI argument combination was exercised with real handoffs and
recipient pickup in Claude Code 2.1.x. Use `subprocess.run` with an argument list
and the prepared text on stdin, so handoff contents are never shell code:

```python
argv = ["claude", "-p", "--no-session-persistence", "--tools", "SendMessage",
        "--strict-mcp-config", '{"mcpServers":{}}',
        "--system-prompt", "Relay the authorized message exactly; no implementation or settings changes.",
        "--output-format", "json"]
prompt = ("Use SendMessage exactly once to " + verified_socket +
          ". Send the following authorized handoff verbatim. Return delivery metadata only.\n\n" +
          prepared_handoff)
result = subprocess.run(argv, input=prompt, text=True, capture_output=True, timeout=90)
```

This launches only a relay; it does not spawn another implementation worker or
change persistent agent configuration. Save and read its exit/result/refusal
receipt. Native enqueue success proves submission, not reading or completion.
If a dependent action needs confirmed pickup, check the tracker or capture the
target once; use the ordinary reason-triggered fallback, not a timer. Never retry
uncertain submission blindly. Preserve drafts throughout.

`notify_when_idle` belongs to the sending Claude session. A short-lived relay
exits, so its subscription is not a durable notification path for the Codex
owner. Use the reply envelope and shared tracker for the actual result. The relay
may inherit Otty lifecycle hook environment; its own lifecycle is not evidence
that the owner's turn ended or that the recipient finished.

This path provides **Codex → Claude** delivery only. A standalone Codex TUI still
has no verified native ingress here: Claude replies via Otty Enter, and Tab may
defer them until the current turn ends. No daemon setting or background watcher
is required or installed by this procedure.
