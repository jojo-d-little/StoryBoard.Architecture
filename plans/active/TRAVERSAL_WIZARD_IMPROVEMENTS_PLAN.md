# Traversal Wizard Improvements Plan

Status: Draft (planning only, no implementation in this slice)
Owner: StoryboardDesigner.App authoring workflows
Last updated: 2026-09-10

## 1. Purpose

Capture a focused follow-on plan for traversal wizard UX and compatibility improvements discovered during portrait-room and area-map testing.

## 2. Problems Observed

1. Wizard apply reload can reset area-map floor context, causing rooms on higher floors to appear missing.
2. Existing-door reuse is strict name-match only (for example Door_W), which can miss valid door-like objects with different names.
3. Wizard door-source selection currently emphasizes object templates, while authoring flow has shifted toward base objects.
4. Reuse behavior is not explicit enough in-row when candidate objects exist but do not match convention.

## 3. Goals

1. Preserve user map context when leaving wizard apply (especially selected floor).
2. Keep naming convention as the zero-friction default.
3. Provide guided fallback for non-convention door objects.
4. Add base-object-first option for creating doors when none are reused.
5. Keep behavior deterministic and testable across reruns.

## 4. Proposed Enhancements

### 4.1 Floor Context Restore After Wizard Apply

1. Snapshot selected area id + selected floor elevation before traversal wizard apply reload.
2. After hierarchy reload/reopen, restore selected floor for that area editor tab.
3. If area not found, gracefully no-op.

Expected impact:
1. Prevents apparent room disappearance after wizard completion.

### 4.2 Door Candidate Discovery and Selection

1. Keep current convention-first lookup (Door_N, Door_E, Door_S, Door_W, etc.).
2. Add secondary detection of likely openable door candidates with non-convention names.
3. Surface per-direction candidate list in wizard row with explicit selection.
4. Let user choose among:
   - reuse convention match
   - reuse selected candidate
   - create new door
   - no door

Candidate scoring heuristics (initial proposal):
1. Openable and lockable.
2. Non-inventoriable preferred.
3. Near expected wall-center for direction.
4. Name/metadata hints increase confidence (contains "door", has traversal-style variables).

### 4.3 Base Object Choice for New Door Creation

1. Add wizard option to choose base object as source for new door creation.
2. Keep existing object-template choice for backward compatibility.
3. Selection precedence for creation path:
   - explicit selected existing candidate
   - explicit selected base object
   - explicit selected object template
   - default internal new door object

### 4.4 Reuse Guidance Messaging

1. Add row-level status text for each direction:
   - exact convention match found
   - non-convention candidates found
   - no candidate found (new door will be created if enabled)
2. Add apply summary counters:
   - reusedByConvention
   - reusedByManualCandidate
   - createdFromBaseObject
   - createdFromTemplate
   - createdDefault

## 5. Risk and Mitigation

Risk: Medium (wizard dialog model/UI + apply path changes).

Mitigations:
1. Stage in small phases with behavior flags if needed.
2. Preserve existing convention-first defaults so old flows still work.
3. Add focused tests before broad manual verification.
4. Keep runtime contracts unchanged (designer-side orchestration only).

## 6. Phased Execution

Phase 1: Floor restore hardening
1. Implement floor snapshot/restore around wizard apply reload.
2. Add focused regression test.

Phase 2: Candidate detection and row model
1. Add candidate discovery service/helper.
2. Extend wizard request/row model with candidate metadata.
3. Add tests for detection confidence and ordering.

Phase 3: Wizard UI selection controls
1. Add per-row candidate dropdown/selection.
2. Add base object chooser for create-new path.
3. Add UX status text and apply summary diagnostics.

Phase 4: Apply pipeline integration
1. Integrate selected candidate/base object/template precedence.
2. Preserve current idempotency for duplicate room-pair handling.
3. Add rerun and migration tests.

## 7. Test Plan (minimum)

1. Floor persists after wizard apply when area rooms are on non-zero floor.
2. Convention-named existing door is auto-selected and reused.
3. Non-convention door candidate can be selected and reused.
4. Candidate list excludes obvious non-door openables under baseline heuristics.
5. New door creation from selected base object works and retains expected transform/variables.
6. Existing duplicate-pair/idempotency wizard tests continue to pass.

## 8. Exit Criteria

1. Wizard no longer causes apparent room disappearance due to floor reset.
2. Authors can reuse non-convention door objects explicitly in wizard.
3. Authors can select base object source for new doors.
4. Convention path remains default and backward compatible.
5. Focused traversal wizard and area-map regressions pass.

## 9. Notes

1. This document captures deferred improvements only.
2. No implementation is included in this planning slice.
