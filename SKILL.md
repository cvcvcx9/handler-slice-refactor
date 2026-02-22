---
name: handler-slice-refactor
description: Refactor feature code by splitting event/side-effect logic into focused handlers and separating Zustand state into concern-based slices while preserving runtime behavior. Use for requests like "핸들러 분리", "slice 분리", "가독성 개선", "관심사 분리".
---

# Handler + Slice Refactor

Use this skill when a feature became hard to read because routing logic, side effects, API calls, and state transitions are mixed in one place.

## Goals

- Keep behavior the same while making intent obvious
- Separate orchestration from business logic
- Separate Zustand state by concern (session, realtime, hints, analysis, logs)
- Keep verification strict (`lsp_diagnostics` + lint + build)

## When to Trigger

- "핸들러 분리해줘"
- "slice로 관심사 분리해줘"
- "조건문 많아서 읽기 힘들다"
- "eventId/responseId 의미가 안 보인다"

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
