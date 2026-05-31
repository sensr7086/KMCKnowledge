---
title: "UMCNiagaraSocketBindings"
kind: entity
category: AssetUserData
base_class: UAssetUserData
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/Actor/AssetData/MCNiagaraSocketBindings.h
  - KMCProject/MCPlayModule/Actor/AssetData/MCNiagaraSocketBindings.cpp
vault_refs:
  - sources/ue-assetclasses-assetuserdata
  - sources/ue-niagara-skill
  - concepts/Asset-Loading-Policy
  - concepts/Editor-Only-4-Tier-Separation
  - synthesis/editor-preview-scene-runtime-handoff
  - sources/ue-editor-propertyeditor
last_ingested: 2026-05-29
---

# UMCNiagaraSocketBindings

## 한 줄 정의
StaticMesh / SkeletalMesh 의 socket 에 Niagara 를 attach 하는 메타데이터(UAssetUserData) — UE Mesh Editor 자체 확장(엔진 수정 X). 디자이너가 Details > Asset User Data 에서 +Add 로 정의.

## 소속 / 상속
- 대분류: **DataTable** (에셋 류 — edge-cases.md "보통 DataTable 대분류에 준해 처리"). 단, **본 자산은 Component 와도 강하게 결합** — [[entities/MCSoftStaticMeshComponent]] / [[entities/MCStaticMeshNiagaraSpawnerComponent]] 의 런타임 소비, [[entities/MCNiagaraSocketPreviewSubsystem]] 의 Editor preview.
- `UCLASS(DefaultToInstanced, EditInlineNew, CollapseCategories, BlueprintType, meta=(DisplayName="MC Niagara Socket Bindings"))`
- `class MCPLAYMODULE_API UMCNiagaraSocketBindings : public UAssetUserData`
- 함께 정의:
  - `USTRUCT(BlueprintType) FMCNiagaraSocketBinding` — Socket Name + Niagara Soft + Offset + Activate 옵션 + Debug 색·반경.
  - `DECLARE_MULTICAST_DELEGATE_OneParam FMCOnNiagaraSocketBindingsChanged(UObject* HostAsset)` — static 멀티캐스트, MCPlayModule → MCEditorModule backward dep 회피용.

## 핵심 구조
- **UPROPERTY**:
  - `TArray<FMCNiagaraSocketBinding> Bindings` — 디자이너가 +Add 로 채움.
- **FMCNiagaraSocketBinding 필드**:
  - `FName SocketName` — Mesh 의 socket 이름. Details Customization 이 실제 socket 목록 드롭다운.
  - `TSoftObjectPtr<UNiagaraSystem> NiagaraSystem` — Soft 권장 (첫 Spawn 히칭 회피). 비어있으면 skip.
  - `FVector LocationOffset / FRotator RotationOffset / FVector Scale=FVector::OneVector`
  - `bool bAutoActivate=true`
  - `FLinearColor DebugColor` (시안 디폴트) + `float DebugSphereRadius=8.f` (1~100) — Editor viewport 디버그.
- **`#if WITH_EDITOR`**:
  - `static FMCOnNiagaraSocketBindingsChanged OnBindingsChanged` — Editor 서브시스템 구독 hook.
  - `virtual void Draw(FPrimitiveDrawInterface*, const FSceneView*) const override` — Stage 3, UE 가 자산 Editor viewport 에서 자동 호출. Socket 위치에 디버그 sphere + 라벨.
  - `virtual void PostEditChangeProperty(FPropertyChangedEvent&) override` — Bindings 수정 시 PreviewSubsystem refresh 통지.
  - `virtual void PostEditUndo() override` — 🆕 Undo/Redo 후 자동 호출. OnBindingsChanged.Broadcast → PreviewSubsystem + SMCBindingsListWidget 자동 갱신.

## 따르는 패턴
- UCLASS specifier 의미: DefaultToInstanced(자동 인스턴스화), EditInlineNew(+Add 메뉴 노출), CollapseCategories(디테일 패널 카테고리 접힘) → [[sources/ue-assetclasses-assetuserdata]] §4 UAssetUserData 발자국 패턴
- StaticMesh / SkeletalMesh 양쪽 호환 — 둘 다 IInterface_AssetUserData 구현 → [[sources/ue-assetclasses-assetuserdata]] §3 표
- 자주 사용해도 Soft 권장 (Niagara System 첫 Spawn 히칭 회피) → [[concepts/Asset-Loading-Policy]] §1
- UAssetUserData::Draw 가상 hook (Editor only) → [[concepts/Editor-Only-4-Tier-Separation]] / [[sources/ue-assetclasses-assetuserdata]] §1
- SpawnSystemAttached + ENCPoolMethod::AutoRelease → [[sources/ue-niagara-skill]]
- RefreshForAsset 패턴 (디테일 변경 → PreviewSubsystem refresh) → [[synthesis/editor-preview-scene-runtime-handoff]]
- PostEditUndo + Modify() 페어 (Undo/Redo 표준) → [[sources/ue-editor-propertyeditor]] §함정 1
- 디자이너 워크플로:
  1. Mesh 더블클릭 → UE Mesh Editor
  2. Details → "Asset User Data" 카테고리 (UE 기본)
  3. + Add → "MC Niagara Socket Bindings"
  4. Bindings 배열 채우기 (SocketName + NiagaraSystem + Offset)
  5. Save → Mesh.uasset 안 메타 영구 저장
- backward dep 회피: MCPlayModule(런타임 자산) ← Editor Subsystem 이 static delegate 로 구독. 자산이 Editor 모듈을 직접 의존 안 함.

## ⚠ 함정
- **NiagaraSystem 빈 binding 은 skip**: 디자이너가 +Add 후 NiagaraSystem 채우지 않으면 런타임/Preview 양쪽 모두 무시.
- **Debug 의 ClampMin=1.0 / ClampMax=100.0**: 0 또는 100 초과 시 ClampMax 적용.
- **static OnBindingsChanged 의 라이프사이클**: static 이므로 PIE 종료·모듈 reload 후에도 구독자 누적 가능 — Editor Subsystem 이 Deinitialize 에서 Remove 페어 의무.
- **WITH_EDITOR 가드**: Draw / PostEditChangeProperty / PostEditUndo / OnBindingsChanged 모두 Editor only. Cooked 빌드는 디버그 표시·refresh 없음.

## 연관 entity
- [[entities/MCStaticMeshNiagaraSpawnerComponent]] — BeginPlay 시 본 자산 메타 읽고 SpawnSystemAttached (런타임).
- [[entities/MCSoftStaticMeshComponent]] — Soft 메시 로드 후 본 자산 메타 추출 + Niagara spawn (Soft 경로).
- [[entities/MCSoftSkeletalMeshComponent]] — Skeletal 측 동일 가능성 (🟡 — 헤더에서 직접 확인 안 됨, AssetUserData 표준상 호환).
- [[entities/MCNiagaraSocketPreviewSubsystem]] — Editor preview + Nomad 도킹 탭.
- MCNiagaraSocketBindingHelpers (미인덱싱 — 공통 헬퍼).
- SMCBindingsListWidget (MCEditorModule — 미인덱싱).
