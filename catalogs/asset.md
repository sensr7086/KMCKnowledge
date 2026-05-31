---
title: "Asset 대분류"
kind: catalog
category: Asset
last_ingested: 2026-05-29
---

# Asset 대분류

> KMCProject 의 *primary* 자산 — UObject 자산 콘텐츠 호스트 + UDataAsset / UPrimaryDataAsset (DataTable 자손 제외). UAssetUserData(에셋 메타)는 [[catalogs/assetuserdata]] 분리. **카테고리 도입**: 2026-05-29 (배치 4 Story 직후).

## UObject 자산 (그래프 호스트)
- 🟡 [[entities/MCStoryAsset]] — Story 그래프 자산 호스트.
- 🟡 [[entities/MCPartsAsset]] — Parts 그래프 자산 호스트.

## UDataAsset / UPrimaryDataAsset
- 🟡 [[entities/MCStoryBoard]] — Story 캐릭터 바인딩 (Abstract Blueprintable, placeholder 수준).
- 🟢 [[entities/MCComboAsset]] — Combo 시계열 자산 (LevelSequence MovieScene 미러). 5단계 계층 + Phase 4c 마이그레이션 + Phase 7a 평가 entry + Phase 6g Asset-level Tracks.
- 🔴 [[entities/MCMeshProxyDataAsset]] — `UPrimaryDataAsset` (의도). **본문 주석 처리 placeholder** ([[entities/MCMeshProxyUserAssetData]] 페어).
- 🟢 [[entities/MCLootTableAsset]] — `UDataAsset`. 단일 루팅 테이블 (table_id/rolls/empty_chance + EntryRowNames 큐레이션 부분집합). **CP-1 Option B**: 값은 공용 DT, 멤버십은 EntryRowNames — DT 경로 이중화 회피.

## 페어 카탈로그
- [[catalogs/graphnode]] — 자산이 보관하는 노드 컬렉션.
- [[catalogs/subsystem]] — 자산을 호스팅하는 Subsystem.
- [[catalogs/component]] — 런타임 자산 소비 Component ([[entities/MCPartsLoaderComponent]]).
- [[catalogs/interface]] — Asset 측 visitor / 평가 인터페이스 ([[entities/MCComboPreviewVisitor]]).
- [[catalogs/assetuserdata]] — 자산에 부착되는 메타.

## 공통 함정
- **Hard 자산 사이클**: 자산 → 자산 Hard 참조 시 사이클 가능 — 디자이너 의무. Story 의 ActionRunStroy 예시.
- **Soft 권장**: TSoftObjectPtr 권장 → [[concepts/Asset-Loading-Policy]] §1. Cooked 첫 SpawnActor 히칭 4단 회피.
- **NodeMap / 런타임 컬렉션 reconstructed**: post-load 시점 RecursiveChildNode 호출 전 invalidate.
- **abstract DataAsset 자손 의무**: 직접 인스턴스화 차단, BP 자손 필수.
- **자산 멤버의 raw `AActor*`**: 라이프사이클 mismatch 가능.
- **Preview 멤버의 Cooked 포함**: `PreviewMesh / PreviewAnimation` 같은 에디터 전용 멤버는 #if WITH_EDITORONLY_DATA 가드 권장. MCPartsAsset 미적용.
- **Graph raw 포인터**: TObjectPtr 미적용 UE 5.0+ 권장 패턴.
- **_DEPRECATED UPROPERTY + PostLoad 마이그레이션 패턴** (MCComboAsset Phase 4c): meta=(DeprecatedProperty) + PostLoad 자동 이전 + Modify 마크. Engine MovieScene.h Possessables_DEPRECATED 패턴 미러. 마이그레이션 1회 후 빈 배열 직렬화.
- **CallInEditor meta + BlueprintCallable**: Detail Panel 안 버튼 노출 + 런타임 BP 그래프 표시 X (Trap 30 cleanup).
- **시간 jump 함정 (스크럽)**: Phase 7a evaluation 의 transition 검출은 통과 Section 미발화. Sequencer 표준은 동시 호출. 향후 권고.
