# Compatibility

**Snapshot:** `v0.3.2-alpha`
**Last updated:** 2026-03-29

This file tracks real software validation progress for silex.

## Status Legend

- `Boots` - reaches executable game code and progresses past reset/startup
- `In Intro` - reaches title / intro / non-gameplay scenes with meaningful rendering
- `Partial Gameplay` - enters gameplay or gameplay-adjacent scenes with known issues
- `Blocked` - currently blocked by a specific subsystem gap
- `Validated` - targeted subsystem/test passes its intended milestone

## Commercial Library

| Title | Status | Current Result | Primary Remaining Issues |
|---|---|---|---|

## Validation Games

| Game / Suite | Status | Result |
|---|---|---|
| PeterLemon CPU test suite | Validated | Current CPU validation baseline |

## Notes

## Compatibility Workflow

When adding a new title:

1. record the first reliable milestone reached
2. record the specific blocker, not a vague symptom
3. update the title only when the blocker changes materially
4. avoid inflating compatibility claims from one-off scenes or accidental rendering
