# Host Schema One-Shot Warning Checklist

Generated: 2026-08-14

## Goal
- Remove HostContracts CS8618 warnings from generated DTOs in one batch.
- Keep behavior unchanged unless explicitly noted.

## Edit Rules (One Pass)
- For optional properties (not in required list): prefer nullable emitted C# types.
- For required complex properties (object/array): set a deterministic generated initializer where contract-safe.
- Do not change required lists unless you intentionally want stricter/looser contract behavior.

## Warning Buckets And Schema Targets

### 1) Context Properties (CS8618)
Pattern: non-nullable `Context` emitted without initializer.

Schemas:
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostDiscoverGamesRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostGetGameDetailsRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostAssetManagementDtos/HostDiscoverAssetsRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostAssetManagementDtos/HostGetAssetRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostAttachSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostEndSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostGetSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostJoinSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostLeaveSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostListSessionsRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostStartSessionRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionPersistenceDtos/HostCaptureSessionStateRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionPersistenceDtos/HostLoadSessionStateRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostAuthenticateRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostGetCurrentPrincipalRequest.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostLogoutRequest.schema.json

Suggested treatment:
- Keep required semantics.
- Emit with safe default initializer for context refs where that matches current hand-authored expectation.

### 2) Result Properties (CS8618)
Pattern: non-nullable `Result` emitted without initializer.

Schemas:
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostDiscoverGamesResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostGetGameDetailsResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostAssetManagementDtos/HostDiscoverAssetsResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostAssetManagementDtos/HostGetAssetResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostContractDtos/HostCommandProcessedEventArgs_contract.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostAttachSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostEndSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostGetSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostJoinSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostLeaveSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostListSessionsResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostStartSessionResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionPersistenceDtos/HostCaptureSessionStateResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionPersistenceDtos/HostLoadSessionStateResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostAuthenticateResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostGetCurrentPrincipalResult.schema.json
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/UserManagementDtos/HostLogoutResult.schema.json

Suggested treatment:
- Keep required semantics.
- Emit with safe default initializer for result envelope refs.

### 3) Nested Descriptor Refs (CS8618)
Pattern: required nested object refs emitted non-nullable with no ctor value.

Schemas:
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/SessionManagementDtos/HostSessionDescriptor.schema.json
	- joinPolicy
	- access
	- membership
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostGameDescriptor.schema.json
	- access
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/GameDiscoveryDtos/HostGameDetailsDescriptor.schema.json
	- access

Suggested treatment:
- Keep required semantics.
- Add deterministic defaults for nested descriptors or make nullable only if contractually optional.

### 4) Required Payload/Object/Collection Members (CS8618)
Pattern: non-null member with no generated default.

Schemas:
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostContractDtos/HostCommandPresentationCue_contract.schema.json
	- parameters
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostAssetManagementDtos/HostGetAssetResult.schema.json
	- payloadBytes
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostContractDtos/HostCommandRenderableRoomObject_contract.schema.json
	- renderableImage
- Storyboard.Shared.Contracts/HostContracts/Schemas/Dtos/HostContractDtos/HostCommandRoomDirectionalImage_contract.schema.json
	- renderableImage

Suggested treatment:
- If required, emit deterministic defaults (e.g., empty byte array, empty dictionary, default nested instance as appropriate).

## Suggested Execution Order
1. Batch apply context + result treatment first (largest reduction).
2. Apply nested descriptor fixes.
3. Apply payload/object/collection defaults.
4. Regenerate host staging.
5. Lock host contracts.
6. Rebuild and re-audit warnings.

## Non-Schema Warning (Code)
- Storyboard.Simulator/ViewModels/SimulatorViewModel.cs line 2551 (CS8601)

## Validation Commands
- dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- generate-host-contract-staging
- dotnet run --project .\Storyboard.SchemaCodegen\Storyboard.SchemaCodegen.csproj -- lock-host-contract
- dotnet build .\StoryboardDesigner.slnx

