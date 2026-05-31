---
title: "AssetUserData 대분류"
kind: catalog
category: AssetUserData
last_ingested: 2026-05-29
---

# AssetUserData 대분류

> KMCProject 의 `UAssetUserData` 자손. Mesh / Skeleton / 기타 자산에 첨부되는 메타 데이터. **카테고리 도입**: 2026-05-29 (배치 3 직후) — DataTable 강제 배치 해소.

## Niagara / VFX
- 🟢 [[entities/MCNiagaraSocketBindings]] — `UAssetUserData`. Mesh socket 의 Niagara binding 목록 (`TArray<FMCNiagaraSocketBinding>`). StaticMesh / SkeletalMesh 양쪽 호환. Editor +Add 메뉴 노출, Stage 3 Draw 가상 hook, Undo/Redo 자동 broadcast.

## Animation / Hit Reaction
- 🟢 [[entities/MCHitBoneCurveUserData]] — `UAssetUserData`. SkeletalMesh 본별 히트 회전 커브 (`UCurveVector` 어셋 1개로 3축 통합). Phase 4 Editor "Validate Data" 통합. dangling pointer 2중 fix.

## Mesh Proxy
- 🔴 [[entities/MCMeshProxyUserAssetData]] — `UAssetUserData` (의도). **본문 주석 처리, 컴파일 비활성 placeholder**. 향후 본문 복구 시 갱신 필요.

## 공통 함정
- **UCLASS 표준 specifier 4종 — 누락 시 Editor +Add 메뉴 미노출**:
  - `DefaultToInstanced` — 자산 안 자동 인스턴스화.
  - `EditInlineNew` — 디테일 패널 +Add 메뉴 노출.
  - `CollapseCategories` — 디테일 패널 카테고리 접힘 (선택).
  - `BlueprintType` — BP 노출.
  → [[sources/ue-assetclasses-assetuserdata]] §4.
- **`UAssetUserData::Draw` 가상 hook (Editor only)**: UE 가 자산 Editor viewport 에서 *자동* 호출 — engine 수정 X. Socket 위치에 디버그 sphere + 라벨. WITH_EDITOR 가드 필수.
- **StaticMesh / SkeletalMesh 양쪽 호환**: 둘 다 `IInterface_AssetUserData` 구현 → 동일 UAssetUserData 가 양쪽 부착 가능. → [[sources/ue-assetclasses-assetuserdata]] §3.
- **TArray UPROPERTY reallocation dangling**: SCurveEditor / IStructureDetailsView 가 raw 포인터 (예: `FRichCurve*`) 캐시 시 stale crash 위험. 표준 fix 2중 — `Reserve(N)` (defense in depth) + view 전체 재생성 (구조적).
- **`UObject::IsDataValid` name hiding (C4264) 회피**: 자손이 동일 시그니처 함수 정의 시 name hiding. BP 헬퍼는 `HasValidXxx` 형태로 분리 (예: MCHitBoneCurveUserData 의 `HasValidBoneCurves`).
- **`PostEditUndo` + `Modify()` 페어**: Undo/Redo 표준. static multicast delegate broadcast 로 외부 구독자 (Editor Subsystem / Slate widget) 자동 갱신. → [[sources/ue-editor-propertyeditor]] §함정 1.
- **`5.6+ PostEditChangeOwner` 시그니처**: 5.x 표준 — `virtual void PostEditChangeOwner(const FPropertyChangedEvent&)` (const ref). 옛 무인자 시그니처와 분리. → [[sources/ue-assetclasses-assetuserdata]] §9 함정 #5.
- **MCPlayModule → MCEditorModule backward dep 회피**: static multicast delegate 패턴 — `UMCNiagaraSocketBindings::OnBindingsChanged` 같이 자산 측에서 static delegate 노출, Editor Subsystem 이 구독. 자산 모듈이 Editor 모듈 직접 의존 안 함.
- **TSoftObjectPtr 권장**: 자주 사용해도 첫 Spawn 히칭 회피 → Soft. UCurveVector 같은 외부 자산 참조도 Soft 가능 (현재는 Hard `TObjectPtr` — 본 공유 시 즉시 사용 보장).
