---
title: "Subsystem 대분류"
kind: catalog
category: Subsystem
last_ingested: 2026-05-29
---

# Subsystem 대분류

> KMCProject 의 `USubsystem` 자손. 라이프사이클 범위(GameInstance / World / Editor) 별로 정리. **카테고리 도입**: 2026-05-29 (배치 3 직후) — Actor/Component/DataTable 강제 배치 분류 한계 해소.

## GameInstance Subsystem (게임 진행 ~ 종료, Map 전환 살아남음)
- 🟢 [[entities/MCGameSubsystem]] — `UGameInstanceSubsystem`. UMCTableManager 소유 (Within 페어, 패턴 B). 향후 Save/Load·Online·SeamlessTravel 인계 자리.

## World Subsystem (레벨별, World Travel 시 재생성)
- 🟢 [[entities/MCActorSpawnSubsystem]] — `UWorldSubsystem`. 스폰·풀(`AvailablePoolByClass`)·TOctree2 공간 인덱스·USignificanceManager 통합·콘솔 디버그 명령 4종.

- 🟢 [[entities/MCLootSubsystem]] — `UWorldSubsystem`. 루팅 후보 풀 관리(`RegisteredLootables` TWeakObjectPtr 배열) + `FindClosestLootable` 거리비교(DistSquared 선형 스캔). ShouldCreateSubsystem 으로 Game/PIE 한정. [[entities/MCLootableComponent]] 페어.

## Tickable World Subsystem (World + 매 프레임 Tick)
- 🟡 [[entities/MCStorySubSystem]] — `UTickableWorldSubsystem`. 활성 `UMCStoryAsset` 보관 + 런타임 breakpoint(WITH_EDITOR).
- 🟡 [[entities/MCCameraSubSystem]] — `UTickableWorldSubsystem`. `EMCCameraMode → UMCCameraBase` 맵, 활성 `AMCCamera` 추적, Transform 쿼리.

## Editor Subsystem (MCEDITORMODULE only, WITH_EDITOR 가드)
- 🟢 [[entities/MCNiagaraSocketPreviewSubsystem]] — `UEditorSubsystem`. StaticMesh/SkeletalMesh Editor preview 의 Niagara 자동 spawn + Nomad 도킹 탭 + RefreshForAsset.

## 공통 함정
- **PIE 다중 인스턴스 race**: Server PIE / Client PIE 각자 별도 GameInstance / World → 별도 Subsystem 인스턴스. 글로벌 mutate 시 race 가능. → [[synthesis/subsystem-5-types-decision-tree]] §6.
- **`IsTickable() const { return true; }` 무조건 활성**: ActivationStory / CurrentMode == None 같은 비활성 상태에도 Tick 발화 → Tick 본체에서 가드 의무 (MCStorySubSystem, MCCameraSubSystem 양쪽).
- **`Deinitialize` 의 역순 강제**: Initialize 의 역순(자식 → 자기 → Super::Deinitialize) — MCGameSubsystem 의 TableManager 정리 패턴.
- **Pimpl + `TUniquePtr<IncompleteType>` C4150 회피**: MCActorSpawnSubsystem 의 `TPimplPtr<FOctreeData>` 패턴. UE `TPimplPtr<>` 는 MakePimpl<T>() 시점에 destroy 함수 capture 해 incomplete 안전.
- **Handle Pin 의무**: `ActiveLoadHandles` / `PreloadHandle` / Niagara `LoadHandle` 미보관 시 GC → 콜백 도착 전 핸들 해제. → [[concepts/Asset-Loading-Policy]] §2 단계 5.
- **Subsystem 라이프사이클 선택 트리** → [[synthesis/subsystem-5-types-decision-tree]] / [[concepts/Subsystem-5-Types]].
- **합성 패턴 B (부모-자식 UObject 소유)**: Subsystem 안에 Subsystem nesting 회피. UMCGameSubsystem → UMCTableManager 가 모범. → [[synthesis/subsystem-advanced-patterns]] §부모-자식 UObject 소유.
- **Editor 분기 — WITH_EDITOR 가드**: Editor only API (OpenCurrentAsset, breakpoint, Draw 등) 는 반드시 `#if WITH_EDITOR` 안. Cooked 빌드 미포함. → [[concepts/Editor-Only-4-Tier-Separation]].
