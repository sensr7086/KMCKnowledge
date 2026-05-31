---
title: "UMCNiagaraSocketPreviewSubsystem"
kind: entity
category: Subsystem
base_class: UEditorSubsystem
module: MCEDITORMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCEditorModule/Subsystem/MCNiagaraSocketPreviewSubsystem.h
  - KMCProject/MCEditorModule/Subsystem/MCNiagaraSocketPreviewSubsystem.cpp
vault_refs:
  - synthesis/editor-preview-scene-runtime-handoff
  - synthesis/instanced-subobject-customization-bypass
  - sources/ue-editor-editorsubsystem
  - concepts/Editor-Only-4-Tier-Separation
  - concepts/Asset-Loading-Policy
  - concepts/Profiling-Scope-Rule
  - sources/ue-slate-docking
  - sources/ue-editor-eventbinding
last_ingested: 2026-05-29
---

# UMCNiagaraSocketPreviewSubsystem

## 한 줄 정의
Editor 전용 — StaticMesh / SkeletalMesh Editor 가 열릴 때 preview scene 의 메시 컴포넌트에 `UMCNiagaraSocketBindings` 의 Niagara 들을 자동 spawn + Nomad 도킹 탭 + 디테일 변경 RefreshForAsset hook.

## 소속 / 상속
- 대분류: Component (역할 기준 — 메시 컴포넌트 preview 의 Niagara 처리. Editor only.)
- 모듈: **MCEDITORMODULE** (런타임이 아님).
- `UCLASS()` `class MCEDITORMODULE_API UMCNiagaraSocketPreviewSubsystem : public UEditorSubsystem`
- 전체 헤더 본문이 **`#if WITH_EDITOR ... #endif`** 가드 안.
- 함께 정의: `struct FMCNiagaraSocketPreviewEntry` — 자산별 preview 컨텍스트(HostAsset weak, PreviewMesh weak, `TArray<TWeakObjectPtr<UNiagaraComponent>> Spawned`).

## 핵심 구조
- **UEditorSubsystem Interface**:
  - `virtual void Initialize(FSubsystemCollectionBase&) override`
  - `virtual void Deinitialize() override`
- **외부 호출**:
  - `void RefreshForAsset(UObject* HostAsset)` — `UMCNiagaraSocketBindings::PostEditChangeProperty` 에서 디테일 변경 통지. 열린 자산이면 destroy + 재 spawn.
  - `static UMCNiagaraSocketPreviewSubsystem* Get()` — 정적 편의.
- **private 헬퍼 (preview 라이프사이클)**:
  - `void OnAssetOpenedInEditor(UObject* Asset, IAssetEditorInstance*)`
  - `void OnAssetEditorRequestClose(UObject* Asset, EAssetEditorCloseReason)`
  - `UMeshComponent* ResolvePreviewMesh(UObject* Asset, IAssetEditorInstance*) const` — StaticMesh / SkeletalMesh 분기.
  - `void SpawnForEntry(FMCNiagaraSocketPreviewEntry&)` — WorldType 검증 + LoadAllSynchronous + SpawnAllNow.
  - `void DestroyForEntry(FMCNiagaraSocketPreviewEntry&)` — DeactivateAll (bForceDestroy=true). Load Pin handle 없음.
- **Entry 매핑**:
  - `TMap<TWeakObjectPtr<UObject>, TUniquePtr<FMCNiagaraSocketPreviewEntry>> Entries` — TUniquePtr 로 TMap 재할당 시 raw 포인터 무효화 회피 (콜백이 entry 주소 캡처).
- **델리게이트 핸들**: `OpenedHandle`, `CloseHandle`, `BindingsChangedHandle`, `PostEngineInitHandle` (도킹 탭 시점 지연 — Slate 미초기화 시 hook).
- **도킹 탭 (Phase 4.3 우회 c)**:
  - `static const FName MCBindingsTabId` — 전역 유일 Tab Spawner FName.
  - `void RegisterDockTab()` — Initialize 안 (FGlobalTabmanager Nomad 탭 등록).
  - `void UnregisterDockTab()` — Deinitialize 안 페어.
  - `TSharedRef<SDockTab> SpawnDockTab(const FSpawnTabArgs&)` — Tab Spawner 콜백. P1: placeholder. P2: SMCBindingsListWidget 임베드.
  - `TWeakObjectPtr<UObject> CurrentEditedAsset` — HandleAssetOpened/Close 가 캐싱. Tab Spawn 시 표시 대상.
  - `TWeakPtr<SMCBindingsListWidget> WeakListWidget` — P3, SetBindings(...) 자동 갱신.
  - `UObject* ResolveCurrentEditedAssetForDockTab() const` — nullptr 면 `GetAllEditedAssets` 로 강제 검색.
  - `void SyncDockTabBindings()` — 자산 변경 시 SetBindings 헬퍼.

## 따르는 패턴
- 동작 (헤더 주석):
  1. AssetEditor 열림 (UStaticMesh/USkeletalMesh) → preview mesh 컴포넌트 획득.
  2. 메시 자산의 `GetAssetUserData<UMCNiagaraSocketBindings>` 추출.
  3. WorldType (Editor/EditorPreview) 검증 → `MCNiagaraSocketBindingHelpers::LoadAllSynchronous` → 즉시 `SpawnAllNow`.
  4. AssetEditor 닫힘 → DeactivateAll (bForceDestroy=true).
- 동일 헬퍼(MCNiagaraSocketBindingHelpers) 재사용 — 런타임 [[entities/MCStaticMeshNiagaraSpawnerComponent]] / [[entities/MCSoftStaticMeshComponent]] 와 DRY.
- Editor preview scene runtime handoff 패턴 (패턴 A) → [[synthesis/editor-preview-scene-runtime-handoff]]
- Phase §4.3 우회 c (Nomad 탭 — 사용자가 SM 에디터 옆에 도킹 자유 결정) → [[synthesis/instanced-subobject-customization-bypass]] §4.3
- §2.6.10 함정 10 차원 2 (자산 에디터 layout delegate 우회) — Nomad 탭은 layout 시스템과 별개로 회피 → [[synthesis/instanced-subobject-customization-bypass]]
- UEditorSubsystem 자손 작성 → [[sources/ue-editor-editorsubsystem]]
- WITH_EDITOR 가드 → [[concepts/Editor-Only-4-Tier-Separation]]
- Editor Pure = Sync (LoadSynchronous) 표준 → [[concepts/Asset-Loading-Policy]] §3
- TRACE_CPUPROFILER_EVENT_SCOPE 의무 → [[concepts/Profiling-Scope-Rule]]
- Nomad 탭 RegisterNomadTabSpawner 패턴 (verified) → [[sources/ue-slate-docking]] §3.1
- OnAssetOpenedInEditor 2-param + 라이프사이클 (verified) → [[sources/ue-editor-eventbinding]]
- Async FStreamableHandle 의존 제거 — 런타임 spawner 측만 유지. SpawnSystemAttached 가 UNiagaraComponent 안에서 NiagaraSystem strong-hold → GC 회피.

## ⚠ 함정
- **§7 함정 1 (vault) — Tab Spawner FName 전역 유일**: `MCBindingsTabId` 가 전역 unique. 충돌 시 Nomad 탭 등록 실패. → [[sources/ue-slate-docking]] §7.
- **WorldType (Editor/EditorPreview) 검증 의무** — 런타임 분기 컨텍스트에서 본 Subsystem 의 spawn 경로 사용 금지. PIE/Cooked Game 은 [[entities/MCStaticMeshNiagaraSpawnerComponent]] 또는 [[entities/MCSoftStaticMeshComponent]] 분기 사용.
- **PostEngineInitHandle 지연 등록**: Slate 가 Initialize 시점에 미초기화 가능 → PostEngineInit hook 으로 지연 등록.
- **TMap 재할당 raw 포인터 무효화 회피**: TUniquePtr 로 감싼 Entries — 콜백이 entry 주소 캡처해도 안전.
- **Async Pin handle 의도적 미보관**: Editor Pure 컨텍스트 = LoadSynchronous 표준 ([[concepts/Asset-Loading-Policy]] §3). 런타임 측만 §3 PIE 분기로 Async + Handle Pin.

## 연관 entity
- [[entities/MCSoftStaticMeshComponent]] — 동일 헬퍼 재사용 (런타임 측 Async path).
- [[entities/MCStaticMeshNiagaraSpawnerComponent]] — 동일 헬퍼 재사용 (런타임 측 페어).
- UMCNiagaraSocketBindings (미인덱싱 — AssetUserData 배치, 자산 메타 정의).
- MCNiagaraSocketBindingHelpers (미인덱싱 — 공통 헬퍼).
- SMCBindingsListWidget (MCEditorModule — 미인덱싱, 도킹 탭 안 SListView).
