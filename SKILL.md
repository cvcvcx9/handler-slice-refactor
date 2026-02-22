---
name: handler-slice-refactor
description: A simple, behavior-safe solution for complex event/state code: split event and side-effect logic into focused handlers and separate Zustand state into clear concern-based slices.
---

# Handler + Slice Refactor

Use this skill when a feature became hard to read because routing logic, side effects, API calls, and state transitions are mixed in one place. It provides a simple path to improve readability without changing runtime behavior.

## Goals

- Keep behavior the same while making intent obvious
- Separate orchestration from business logic
- Separate Zustand state by concern (session, realtime, hints, analysis, logs)
- Keep verification strict (`lsp_diagnostics` + lint + build)

## When to Trigger

- "Split this into handlers."
- "Separate this into concern-based slices."
- "This function has too many conditions and is hard to read."
- "eventId/responseId intent is unclear."
- "Refactor for readability without changing behavior."

## Refactor Rules

### 1) Event Handler Rules

- Keep top-level handler as router/orchestrator only
- Prefer `switch(event.type)` over long `if` chains
- Extract tiny helpers with intent-revealing names:
  - `getResponseId(...)`
  - `resolvePurposeMeta(...)`
  - `applyPendingBootstrapActions(...)`
- Isolate side-effects into dedicated handlers:
  - enrichment request handler
  - hint request handler
  - session end handler

### 2) Zustand Slice Rules

- One slice = one concern
- Put API payload shaping + response normalization in the same concern slice
- Keep `store.ts` composition explicit (`createXSlice`)
- Keep `types.ts` interfaces aligned with slice boundaries

### 3) Naming Rules

- Avoid vague names (`id`, `data`, `temp`)
- Use semantic names:
  - `createdResponseId`, `completedResponseId`, `correlationEventId`
  - `lastHintRequestKey`, `requestPurposeByEventId`
- Use action-like function names for side effects:
  - `triggerHintRequestForAssistantUtterance`
  - `requestAssistantEnrichment`

## Before / After Examples

### Handler Split

Before:

```typescript
if (event.type === "response.created") {
  // id extraction + purpose resolution + map mutation + log
}
if (event.type === "response.done") {
  // metadata fallback + side effects + fsm event
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

### Slice Split

Before:

```typescript
// session-slice.ts
// session state + hint request payload + hint response normalize + hint ui state
```

After:

```typescript
// session-slice.ts
// session lifecycle only

// hint-slice.ts
// hint request/normalize/loading/error state only
```

### Directory Structure

Before:

```text
components/voip-v2-test/
  handlers/realtime-event-handler.ts
  slices/session-slice.ts        # session + hint + normalization mixed
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

## Execution Checklist

1. Map current flow (`grep` + `read`) and find mixed responsibilities.
2. Extract handlers first (no behavior change).
3. Extract slice(s) by concern (no behavior change).
4. Rewire store selectors and UI wiring.
5. Ensure duplicate side-effects are deduped with key-based guards.
6. Validate:
   - `lsp_diagnostics` on modified files
   - `yarn lint`
   - `yarn build`

## Anti-Patterns

- Moving logic without preserving call timing (event order regressions)
- Duplicating the same side effect in both transcript and response-done paths
- Keeping request payload/normalize logic scattered across handlers and slices
- Introducing broad rewrites unrelated to target concern

## Output Contract

- Report: what was split, where, and why
- Include file paths for handlers/slices/types/store wiring
- Include verification results (diagnostics/lint/build)
