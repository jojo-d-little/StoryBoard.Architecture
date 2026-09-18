# Small Adventure Tutorial Plan (Implementation-Aligned Refresh)

## Purpose
Keep the tutorial plan aligned to the current Workshop sample implementation so workshop steps, screenshots, and simulator outcomes match what people actually run.

## Source of Truth
This refresh is based on the implemented sample files in [Samples/WorkshopTutorial](Samples/WorkshopTutorial), especially:
1. [Samples/WorkshopTutorial/WorkshopTutorial.sbe.areas/f5b97d334ba242ff8a41129326fd8fc6.area.json](Samples/WorkshopTutorial/WorkshopTutorial.sbe.areas/f5b97d334ba242ff8a41129326fd8fc6.area.json)
2. [Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/698a46f473bc4e9e8333e2ed555417fc.room.json](Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/698a46f473bc4e9e8333e2ed555417fc.room.json)
3. [Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/6a2b490b2f6f45278294976194437296.room.json](Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/6a2b490b2f6f45278294976194437296.room.json)
4. [Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/0f7b95eb27c9475290e9c749c51dd569.room.json](Samples/WorkshopTutorial/WorkshopTutorial.sbe.rooms/0f7b95eb27c9475290e9c749c51dd569.room.json)
5. [Samples/WorkshopTutorial/WorkshopTutorial.sbe.procedures/80423a4c102742479f327762dcee7ad2.procedure.json](Samples/WorkshopTutorial/WorkshopTutorial.sbe.procedures/80423a4c102742479f327762dcee7ad2.procedure.json)

## What Changed vs Original Write-Up
1. The project now uses 5 rooms, not 4.
2. Key progression is implemented as a procedure-driven repair/mutation flow, not a composite recipe output.
3. Door progression explicitly uses unlock requirements (crowbar for workshop access, brass key for locked exits).
4. Procedure participant requirements are used (including variable requirements on the key state).

## Implemented Adventure Pitch
Short gated hub adventure:
1. Start in Atrium with multiple exits.
2. Supply Closet path is open and contains progression items.
3. Workshop path starts locked and is unlocked by using a crowbar.
4. Brass key starts bent and must be fixed.
5. Fixed key unlocks remaining locked exits from Atrium.
6. Exit Hall is the intended completion destination, and No Where is a secondary/dead-end branch.

## Scope (Current Implementation)
1. 5 rooms.
2. Core interactables: Crowbar, Backpack, Brass Key, Hammer, Anvil, Workbench, directional doors.
3. 1 procedure-based repair flow (`FixKey`).
4. Unlock requirement gates on multiple doors.
5. 10-15 minute first playthrough target remains reasonable.

## Room Plan (As Built)
1. Atrium (start)
2. Supply Closet
3. Workshop
4. Exit Hall
5. No Where (decoy/secondary branch)

## Directional Layout (As Built)
Atrium is the hub:
1. East <-> Supply Closet
2. North <-> Workshop
3. West <-> Exit Hall
4. South <-> No Where

Room placement in map data follows:
1. Atrium at center
2. Supply Closet east
3. Workshop north
4. Exit Hall west
5. No Where south

## Door and Gate Configuration (As Built)
### Atrium doors
1. Door_E to Supply Closet: open, unlocked.
2. Door_N to Workshop: closed, locked; `pry` action unlocks it and requires Crowbar.
3. Door_W to Exit Hall: closed, locked; unlock requires Brass Key.
4. Door_S to No Where: closed, locked; unlock requires Brass Key.

### Key constructs taught here
1. Unlock requirements on lockable objects (`lockOperationRequirements.unlockKeyRequirements`).
2. Custom unlock verb path (`pry`) on Door_N.
3. Shared traversal-facing state variables such as `isOpen` and directional helper values like `lookingFromHere`.

## Item and Tool Setup (As Built)
### Supply Closet
1. Crowbar (inventoriable; heavy enough to highlight carrying choices).
2. Backpack (container; capacity extender behavior).
3. Brass Key (`Brass Key`) with `isBent=true` by default.

### Workshop
1. Hammer (inventoriable).
2. Anvil (in-room fixture).
3. Workbench (in-room fixture/anchor object).

## Synonyms and Command Parsing (As Built)
The sample intentionally uses synonyms on key progression objects.

### Configured synonym examples
1. Brass Key includes synonym: `key`.
2. Workshop Door includes synonyms: `door`, `workshop`.
3. Exit Hall Door includes synonyms: `door`, `exit hall`.

### Player command examples enabled by synonyms
1. `fix key` (instead of requiring exact object name).
2. `open workshop door` (matches both room intent and object type).
3. `pry workshop` (still resolves to the workshop door object).
4. `unlock exit hall door with key` (natural phrase form).

### Why this helps gameplay
1. Reduces parser friction by accepting natural player wording.
2. Makes progression verbs easier to discover during first playthrough.
3. Keeps players focused on puzzle intent rather than exact naming.
4. Improves tutorial teachability because multiple command phrasings can be demonstrated and all succeed.

### Authoring guidance for synonym quality
1. Add at least one generic synonym (`door`, `key`) and one context synonym (`workshop`, `exit hall`).
2. Keep synonyms short and unambiguous within the current room.
3. Avoid overloading the same synonym on multiple high-priority objects in one scope.
4. Validate with at least two alternate command phrasings per gated step.

## Key Repair Flow (Procedure, Not Composite)
### Procedure
1. Procedure name: `FixKey`.
2. Triggered by Brass Key action `fix` (`InvokeProcedure`).
3. Success output uses procedure description text.

### Participant requirements
1. Brass Key: explicit mention required, quantity 1, with variable requirement `isBent == true`.
2. Anvil: presence required.
3. Hammer: presence required.

### Mutation
1. On success, procedure sets Brass Key `isBent=false`.
2. This updates state/visual meaning of the key and allows progression messaging to reflect repaired state.

## Inventory Capacity Demo (As Built)
1. Backpack uses container point variables (`containerPoints`, `containerPointsRemaining`, `capacityPointShareDivider`).
2. Crowbar is configured heavier than small items, making container use meaningful.
3. Recorded simulator flow demonstrates placing Crowbar in Backpack before continuing.

## Minimal Command Flow (Implementation-Aligned)
1. `go east`
2. Acquire or stow tooling in closet (for example `put crowbar into backpack`, then collect key)
3. `go west`
4. `pry workshop door`
5. `open workshop door`
6. `go north`
7. Ensure requirements are present, then `fix brass key`
8. Return to Atrium
9. Unlock/open west door with Brass Key
10. `go west` to Exit Hall

Optional branch:
1. Unlock/open south door and visit No Where.

## Build-Today Checklist (Updated to Match Sample)
1. Create 5 rooms and place them in the hub-plus-cross layout.
2. Wire 4 Atrium traversal pairs (E/N/W/S).
3. Configure door states and lock requirements exactly as implemented.
4. Add Crowbar, Backpack, and Brass Key to Supply Closet.
5. Add Hammer, Anvil, and Workbench to Workshop.
6. Add `fix` action on Brass Key as `InvokeProcedure`.
7. Add `FixKey` procedure with participant requirements and one mutation (`isBent=false`).
8. Verify Exit Hall and No Where are both gated behind repaired Brass Key path.
9. Run simulator walkthrough from start through Exit Hall.

## Quick Acceptance Criteria (Updated)
1. Player reaches Supply Closet immediately from Atrium east.
2. Workshop path cannot progress until Door_N is unlocked via `pry` and Crowbar requirement.
3. `fix brass key` fails without required participants or when key is already fixed.
4. `fix brass key` success flips `isBent` to false.
5. Atrium west/south locked exits require Brass Key-based unlock.
6. Exit Hall is reachable only after intended progression chain.
7. Synonym command variants for key and doors resolve correctly in simulator.

## Grow-Into-Tutorial Staging (Refreshed)
### Stage 1: Layout and Traversal
1. Build 5-room cross layout and directional traversals.
2. Validate directional names and traversal behavior.

### Stage 2: Door State and Unlock Requirements
1. Configure open/locked defaults.
2. Add unlock requirement gates on doors.
3. Demonstrate custom unlock verb (`pry`) on one door.

### Stage 3: Inventory and Carrying Pressure
1. Add Crowbar, Backpack, and key.
2. Demonstrate carrying/container behavior during progression.

### Stage 4: Procedures and Participant Requirements
1. Add `FixKey` procedure.
2. Teach explicit mention vs presence requirements.
3. Teach variable requirement (`isBent`) and mutation (`isBent=false`).

### Stage 5: Objective Completion
1. Reuse repaired key to unlock final path.
2. Reach Exit Hall as completion checkpoint.

## Optional Next Enhancement (Explicitly Not in First Pass)
If you want to bring composite authoring back into this tutorial later:
1. Keep current procedure-driven fix path as baseline.
2. Add a follow-on chapter that replaces or parallels it with a composite recipe path.
3. Compare procedure requirements vs composite part requirements as a teaching moment.

## Notes for Tutorial Authors
1. Keep stage outputs runnable and deterministic.
2. Use stable names already present in the sample to minimize mismatch risk.
3. Call out advanced constructs explicitly: unlock requirements and procedure participant requirements.
4. If gameplay drift is introduced later, update this plan first, then screenshots/scripts.
