# Top-Left Anchor Simplification - Phase 0 Baseline Packet

Date: 2026-07-23

## Scope
Baseline capture only.
No behavior/placement logic changes in this phase.

## Gate Results
1. Build gate
- Command: dotnet build .\StoryboardDesigner.slnx
- Result: Passed
- Note: 3 existing xUnit analyzer warnings in StoryboardDesigner.App.Tests (ActionIntegrityRulesTests xUnit2031)

2. Playback smoke gate
- Command: dotnet test .\StoryboardDesigner.App.Tests\StoryboardDesigner.App.Tests.csproj --filter "FullyQualifiedName~GameSimulatorPlaybackRegressionTests.Replay_RecordedSession_OutputLinesMatchAtEachStep"
- Result: Passed
- Summary: total 6, failed 0, succeeded 6, skipped 0

## Baseline Visual Observation (User)
1. N and S doors: slightly too low.
2. E and W doors: slightly too far right.
3. Overall: close, but systematic directional drift.

## Baseline Tuple Packet (Latest Provided)
[RENDER TUPLE] src=roomChangePayload; name='Door_N'; id=d1f9944a69854cb3a659dc3b2253a57a; runtimeRaw=(280,0); runtimeFinal=(280,7); runtimeAnchor=(280,0); runtimeIconOffset=(0,7); runtimeRot(raw/final)=(0/0); runtimeLocalOffset=(n/a,n/a); payloadAnchor=(280,0); payloadIconOffset=(0,7); payload=(280,7,0,scale=1); hostDraw=(280,7,0,scale=1); img=228x55; pivot=0.5,0.5; selectedPath='assets/images/_shared/12251e687313394ea54d5dd2fbb84926262908bc8e87151e35b8371a37732fae__DarkDoor.png'

[RENDER TUPLE] src=roomChangePayload; name='Door_E'; id=dcc573866deb44448d4761f0774a0bc9; runtimeRaw=(760,200); runtimeFinal=(753,200); runtimeAnchor=(760,200); runtimeIconOffset=(-7,0); runtimeRot(raw/final)=(90/90); runtimeLocalOffset=(n/a,n/a); payloadAnchor=(760,200); payloadIconOffset=(-7,0); payload=(753,200,90,scale=1); hostDraw=(753,200,90,scale=1); img=228x55; pivot=0.5,0.5; selectedPath='assets/images/_shared/4f9c38d990b59e74cc9f31bbd9d0a442d8dfb2162c5fab29600bfd49de99cc3e__DarkDoorOpen.png'

[RENDER TUPLE] src=roomChangePayload; name='Door_S'; id=5980b99eec8b4a6795fd80ff5b66998b; runtimeRaw=(280,560); runtimeFinal=(280,553); runtimeAnchor=(280,560); runtimeIconOffset=(-0,-7); runtimeRot(raw/final)=(180/180); runtimeLocalOffset=(n/a,n/a); payloadAnchor=(280,560); payloadIconOffset=(-0,-7); payload=(280,553,180,scale=1); hostDraw=(280,553,180,scale=1); img=228x55; pivot=0.5,0.5; selectedPath='assets/images/_shared/12251e687313394ea54d5dd2fbb84926262908bc8e87151e35b8371a37732fae__DarkDoor.png'

[RENDER TUPLE] src=roomChangePayload; name='Door_W'; id=7a2d69013ebb42cfb68651509adf6134; runtimeRaw=(0,200); runtimeFinal=(7,200); runtimeAnchor=(0,200); runtimeIconOffset=(7,-0); runtimeRot(raw/final)=(270/270); runtimeLocalOffset=(n/a,n/a); payloadAnchor=(0,200); payloadIconOffset=(7,-0); payload=(7,200,270,scale=1); hostDraw=(7,200,270,scale=1); img=228x55; pivot=0.5,0.5; selectedPath='assets/images/_shared/12251e687313394ea54d5dd2fbb84926262908bc8e87151e35b8371a37732fae__DarkDoor.png'

## Phase 0 Exit Check
1. Baseline build gate captured: Yes.
2. Baseline playback smoke gate captured: Yes.
3. Baseline tuple packet captured: Yes.
4. Baseline directional visual notes captured: Yes.

## Next Phase Readiness
Ready to proceed to implementation phase with phase-scoped rollback and no behavior edits applied during Phase 0.
