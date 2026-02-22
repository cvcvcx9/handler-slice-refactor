# handler-slice-refactor

A simple, behavior-safe skill for improving readability in complex frontend event and state logic.

It helps split:

- mixed event flows into focused handlers
- mixed Zustand state into concern-based slices

without changing runtime behavior.

## When to use

Use this skill when code is hard to follow because event routing, side effects, API calls, and state updates are mixed in one place.

Typical requests:

- "split handlers"
- "separate slices by concern"
- "too many conditions in one function"
- "eventId/responseId intent is unclear"

## What it standardizes

- Event orchestration with `switch(event.type)`
- Intent-first helper naming (`getResponseId`, `resolvePurposeMeta`, etc.)
- Side-effect handlers (`enrichment`, `hint request`, `session end`)
- Concern-based slices (`session`, `realtime`, `hint`, `analysis`, `log`)
- Verification flow (`lsp_diagnostics`, lint, build)

## Install

Global:

```bash
npx skills add cvcvcx9/handler-slice-refactor --skill handler-slice-refactor -g -y
```

Project-level:

```bash
npx skills add cvcvcx9/handler-slice-refactor --skill handler-slice-refactor -y
```

## Files

- `SKILL.md`: main skill definition and workflow
- `README.md`: quick summary and usage context

## Before / After focus

- Before: single file handles routing + metadata resolution + side effects + state mutation
- After: router delegates to small handlers, and state is split into clear slices
Before:
```text
components/voip-v2-test/
  handlers/realtime-event-handler.ts
  slices/session-slice.ts      # session + hint + normalization mixed
  store.ts
  types.ts
```
After:
```text
components/voip-v2-test/
  handlers/
    realtime-event-handler.ts
    realtime-transcription-handler.ts
    realtime-purpose-handler.ts
    realtime-enrichment-handler.ts
    hint-request-handler.ts
  slices/
    session-slice.ts
    hint-slice.ts
    assistant-analysis-slice.ts
    realtime-slice.ts
    log-slice.ts
  ui/
    HintSection.tsx
    AssistantAnalysisSection.tsx
  store.ts
  types.ts
```
## Before / After (Code)
Before:
```typescript
if (event.type === "response.created") {
  // id extraction + purpose resolution + mutation + logging
}
if (event.type === "response.done") {
  // fallback metadata + side effects + fsm event
}
```
After:
```typescript
switch (event.type) {
  case "response.created":
    handleResponseCreatedEvent({ event, setState, getState, addLog });
    return;
  case "response.done":
    handleResponseDoneEvent({ event, setState, getState, addLog });
    return;
}
```
Before:
```typescript
// session-slice.ts
// session lifecycle + hint payload builder + api request + normalization
```
After:
```typescript
// session-slice.ts
// session lifecycle only
// hint-slice.ts
// hint payload + api request + normalization + loading/error state
```
