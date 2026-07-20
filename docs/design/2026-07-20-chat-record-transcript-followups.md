# ChatRecord Transcript Follow-ups

## Context

Post-merge review of the offline ChatRecord projection and providerless
`WebShellTranscript` entry found five bounded edge cases. The existing designs
remain authoritative; this note records the follow-up behavior needed to close
those gaps without expanding the feature.

## Decisions

### Missing tool results

An assistant tool call that reaches replay finalization without a persisted
result remains visible as a synthetic failed tool block. Because the source
history is incomplete, replay also emits a stable, completeness-affecting
diagnostic and the public projection returns `complete: false`.

### Legacy tool errors

Explicit persisted tool-result status and error metadata remain authoritative.
When that metadata is absent, replay inspects the matching `functionResponse`
payload and infers failure from an `error` field or `success: false`. The error
content remains visible. A fully represented failed tool execution does not by
itself make the projection incomplete.

### Conflicting duplicate UUID fragments

The first conversation fragment for a UUID establishes its canonical
`parentUuid`. Later fragments with the same parent continue to aggregate in
source order. A fragment with a different parent produces the existing
`conflicting_parent_uuid` diagnostic but contributes no content or metadata to
the canonical record.

### Providerless status cards

`WebShellTranscript` remains providerless. Serialized task and MCP status
blocks render from their snapshots in read-only mode without polling, daemon
requests, cancellation, reconnect, enable/disable, approval, or authentication
actions. Interactive WebShell surfaces retain their existing behavior.

This is defensive support for the public raw-block boundary. The current
first-party ChatRecord path does not persist or project the task or MCP sentinel
payloads, and this change does not add such persistence.

## Verification

Regression coverage is added at the narrowest public seam for each behavior:

- replay-machine and offline-projection tests for missing and legacy tool
  results;
- transcript-record preparation tests for conflicting UUID fragments; and
- providerless DOM rendering tests for task and MCP status blocks, including
  assertions that no live actions run in read-only mode.

## Out of scope

- Adding daemon providers to `WebShellTranscript`.
- Persisting task or MCP dialog state in ChatRecords.
- Changing valid same-parent fragment aggregation.
- Treating every failed tool execution as incomplete history.
- Refactoring the interactive task or MCP panels beyond the minimum separation
  needed for a static read-only path.
