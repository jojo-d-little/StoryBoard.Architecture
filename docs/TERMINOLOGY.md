# Terminology

Status: Active
Owner: Pending
Last updated: 2026-07-04

Purpose: canonical vocabulary for this repository. Use these definitions in docs, code comments, PR notes, and test naming.

## Core Terms

1. Designer
- Meaning: the authoring host app.
- Project: StoryboardDesigner.App.

2. Simulator
- Meaning: standalone runtime host used to run/load/replay game runtime flows.
- Project: Storyboard.Simulator.

3. Shared Runtime
- Meaning: reusable runtime contracts/services/manager/session logic used by hosts.
- Project: Storyboard.Shared.

4. Native User Project Data
- Meaning: authored game/content data intentionally created by designers.
- Includes: planets/countries/areas/rooms/objects, actions, links, variables, scripts.

5. Native Application State Data
- Meaning: app/session/workspace state, not authored game content.
- Includes: UI selection/tab state, window placement, recent projects.

6. Clean Export
- Meaning: versioned external contract derived from native user project data.
- Rule: must exclude native application/workspace state.

## Object Definition and Instance Terms (Locked)

1. Object Type
- Meaning: canonical reusable definition category.

2. Base Object
- Meaning: concrete canonical definition record used as an instance source.

3. Object Instance
- Meaning: placed object record that references a base object definition.

4. DefinitionId
- Meaning: stable identity link from an instance to its owning base definition.
- Rule: instance linkage persists by DefinitionId, not by display Name.

5. ObjectType
- Meaning: first-class definition-owned type identity field.
- Rules: globally unique across the project; non-overridable on instances.

6. Name (object display name)
- Meaning: human-facing display label.
- Rules: may repeat across scopes; may be locally overridden on instances where allowed.

7. Instanceable
- Meaning: object feature controlling type reuse and multi-instance placement behavior.

8. Quantifiable
- Meaning: separate object feature controlling quantity/count semantics.

9. QuantifiablePlacementDistributionMode
- Meaning: quantifiable-only distribution behavior policy.
- Rule: retained as quantifiable-specific semantics.

10. Instance Overrides
- Meaning: sparse per-instance override payload.
- Rule: inherit-by-default; field diverges only when explicitly overridden.

11. Legacy link field naming
- Legacy: LinkedBaseObjectId.
- Target: DefinitionId.
- Rule: time-bounded migration support only; no legacy naming baggage at final completion.

## Scope Tree Terms

1. Global Scope Node
- Meaning: top-level runtime/project root scope for globally scoped values and vocabulary.

2. Templates Scope Node
- Meaning: container scope for reusable object templates used during authoring.

3. Player Scope Node
- Meaning: scope representing player-owned/runtime-carried object context.

4. Planet Node
- Meaning: top geographic/world hierarchy node beneath global scope.

5. Country Node
- Meaning: world hierarchy node beneath planet, containing area nodes.

6. Area Node
- Meaning: world hierarchy node beneath country; groups rooms and navigation links.

7. Room Node
- Meaning: primary playable location scope where command phrases are routed and room-scoped actions/variables exist.

8. Game Object Node
- Meaning: interactable scoped entity that can exist in rooms or containers and can hold scoped actions/variables.

9. Contained Object Node
- Meaning: game object node nested under another game object via container relationships.

10. Scope Tree
- Meaning: parent/child hierarchy formed by the scope nodes above and attached for deterministic traversal.

## Command and Action Terms

1. Command Phrase
- Meaning: player input routing definition (verb/qualifier) used to resolve actions.
- Scope: currently room-centric in runtime routing flow.

2. Scoped Game Action
- Meaning: executable action definition attached to a scope node.
- Scope support: room and game object scopes (and other scoped contexts as modeled).

3. Linked Action
- Meaning: action-to-action relationship controlling follow-on execution order/conditions.
- Note: linked actions are not room-only; they apply wherever scoped actions exist.

4. Object Commands (legacy/de-emphasized)
- Meaning: object-level command string list field that existed in persistence models.
- Current policy: write-path removed; legacy reads tolerated for compatibility.
- Clarification: this is not the same as scoped object actions or linked actions.

## Runtime Boundary Terms

1. Runtime Scope Node
- Meaning: runtime-facing scope contract node used by preprocess/processor/session flows.

2. Adapter/Bridge
- Meaning: host-side mapping seam from host models/UI context to shared runtime contracts.
- Rule: adapters stay in host projects, not Shared runtime core.

## Runtime Command Parsing Terms

1. Scope Object Token
- Meaning: token parsed from command text that constrains matching to an object scope context.

2. Primary Object Token
- Meaning: first object operand extracted during command preprocessing.

3. Connector Token
- Meaning: grammar connector between object operands (for example transfer-style command phrasing).

## Variable Reference Grammar (Locked)

1. Payload Key Reference
- Meaning: flat key token resolved directly against event payload values.
- Example: procedureId.

2. Anchor-Rooted Reference
- Meaning: explicit anchor-rooted variable reference for scoped/template/event-derived values.
- Syntax: anchorKey::subPropertyKey.variableName.
- Example: currentCommand::primaryCommandObject.isBent.

3. Anchor Root Delimiter
- Token: ::
- Rule: any reference using anchor root must include the delimiter exactly once between anchor and anchored path.

4. Anchored Path Segment Rule
- Rule: anchored path should include subPropertyKey.variableName for event filter authoring.
- Example valid: currentCommand::primaryCommandObject.isBent.
- Example invalid: currentCommand::primaryCommandObject.

5. Legacy Dotted Anchor-Like Path
- Meaning: historical dotted form that omits explicit anchor root.
- Example: primaryCommandObject.isBent.
- Policy: tolerated for compatibility, but considered ambiguous for new authoring and should be upgraded to explicit anchor-rooted form.

## Runtime Action Execution Terms

1. Child Command Forwarding Mode
- Meaning: policy controlling whether/when command handling can be forwarded to child scopes during action resolution.

2. Action Execution Scope
- Meaning: resolved runtime scope context in which a selected action is executed.

3. Command Tick
- Meaning: the bounded runtime execution window for one processed command, including any allowed chained action outcomes.
- Rule: external or concurrent state changes must not interleave mid-tick; only in-tick state mutations from the command's own successful steps are applied between step evaluations.

## Navigation Traversal Terms

1. Traversal Connection
- Meaning: canonical structural navigation relationship between two rooms.
- Rule: one record per unordered room pair in authored model.

2. Traversal Leg
- Meaning: one directed movement opportunity across a traversal connection (A->B or B->A).
- Clarification: use this term instead of "route" for single-step movement.

3. Route (future pathing term)
- Meaning: ordered multi-step sequence of traversal legs.
- Policy: do not use "route" to describe a single directed room-to-room movement.

4. Traversal Direction
- Meaning: compass direction used by navigation movement semantics (for example North, SouthWest).
- Clarification: distinct from grammar "directionals" in command text such as prepositional tokens (for example into, on).

5. Traversal Mode
- Meaning: directional granularity mode controlling allowed movement geometry.
- Values: FourDirectional, EightDirectional.
- Scope model: project default with optional area, room, and traversal-leg overrides.

6. Traversal Access Mode
- Meaning: static directional access policy for a traversal connection.
- Values: TwoWay, OneWayAtoB, OneWayBtoA.

7. Effective Traversal Mode
- Meaning: resolved traversal mode after applying override precedence.
- Precedence: project -> area -> room -> traversal leg.

8. Open State Binding Mode
- Meaning: relationship between side A and side B openable state behavior.
- Values: Independent, SharedWithPairedOpenable.

9. Shared Open State
- Meaning: paired openable behavior where opening or closing either side synchronizes both sides.

## Decision and Change Rules

1. If a term is ambiguous, update this file before broad implementation changes.
2. Prefer one canonical term per concept; list alternates as deprecated wording.
3. If term meaning changes behavior/contract expectations, update tests/docs in same change.

## Deprecated or Avoided Wording

1. "Linked commands" when meaning linked actions.
- Preferred: "linked actions".

2. "Object commands" when meaning scoped object actions.
- Preferred: "scoped object actions" or "object actions".

3. "Connection" when meaning traversal-specific room navigation relationship.
- Preferred: "traversal connection".

4. "Route" when meaning one directed room-to-room movement.
- Preferred: "traversal leg".

5. "Adjacency mode" when meaning FourDirectional/EightDirectional movement constraints.
- Preferred: "traversal mode".

6. "Traversal mode" when meaning TwoWay/OneWayAtoB/OneWayBtoA access policy.
- Preferred: "traversal access mode".

7. "Direction" when movement-specific disambiguation matters.
- Preferred: "traversal direction".
