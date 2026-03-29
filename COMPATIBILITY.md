# Compatibility

**Snapshot:** `v0.3.2-alpha`
**Last updated:** 2026-03-29

This file tracks real software validation progress for silex.

## Status Legend

- `Boots` — reaches executable game code and progresses past reset/startup
- `In Intro` — reaches title / intro / non-gameplay scenes with meaningful rendering
- `Partial Gameplay` — enters gameplay or gameplay-adjacent scenes with known issues
- `Blocked` — currently blocked by a specific subsystem gap
- `Validated` — targeted subsystem/test passes its intended milestone

## Commercial Library

| Title | Status | Current Result | Primary Remaining Issues |
|---|---|---|---|
| Chrono Trigger | Partial Gameplay | Overworld, town/fair scenes, interior rooms, and name entry render with strong scene fidelity | Broader progression and edge-case validation still needed |
| Donkey Kong Country | Partial Gameplay | Title/menu, jungle gameplay, and cave gameplay render correctly | Longer progression and protection-sensitive coverage still needed |
| F-Zero | Partial Gameplay | Title/menu and active race gameplay render correctly, including Mode 7 track scenes at speed | Minor scene artifacting still needs targeted follow-up |
| Final Fantasy IV | Partial Gameplay | Airship scenes, battle UI, dialogue, and castle/interior scenes render correctly | Some transitional black frames still need closer verification |
| The Legend of Zelda: A Link to the Past | Partial Gameplay | Title, file select, name entry, overworld, interiors, and dialogue scenes render correctly | Wider gameplay progression and scene-by-scene validation still needed |
| Super Mario World | Partial Gameplay | Title, intro, world map, and multiple gameplay scenes render correctly at speed | Broader gameplay progression and edge-case validation still needed |
| Super Metroid | Partial Gameplay | Current sweep reaches intro/story, file select, normal gameplay rooms, and escape-sequence gameplay including correct countdown timer and no sprite defects | Wider gameplay progression and edge-case validation still needed |

## Validation Games

| Game / Suite | Status | Result |
|---|---|---|
| PeterLemon CPU test suite | Validated | Current CPU validation baseline |

## Notes

### Current Scope

The current compatibility snapshot reflects active manual validation across the current reference ROM set.

This pass materially improves confidence for:

- Chrono Trigger
- Donkey Kong Country
- F-Zero
- Final Fantasy IV
- The Legend of Zelda: A Link to the Past
- Super Mario World
- Super Metroid

### Known Gameplay-Facing Issues

- F-Zero: core race rendering is strong, but one capture path still shows minor scene artifacting that needs targeted follow-up.

## Compatibility Workflow

When adding a new title:

1. record the first reliable milestone reached
2. record the specific blocker, not a vague symptom
3. update the title only when the blocker changes materially
4. avoid inflating compatibility claims from one-off scenes or accidental rendering
