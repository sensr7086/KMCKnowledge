---
title: "UMCStorySubSystem"
kind: entity
category: Subsystem
base_class: UTickableWorldSubsystem
module: MCPLAYMODULE
confidence: 🟡 PARTIAL
source_paths:
  - KMCProject/MCPlayModule/MCStory/MCStorySubSystem.h
  - KMCProject/MCPlayModule/MCStory/MCStorySubSystem.cpp
vault_refs: []
last_ingested: 2026-05-29
---

# UMCStorySubSystem

## 한 줄 정의
활성 Story Asset 의 라이프사이클·tick·런타임 breakpoint 를 관리하는 World Subsystem. `UMCStoryAsset` 은 자체 tickable 이 아니고 본 Subsystem 경유가 계약.

## 소속 / 상속
- 대분류: **Subsystem** (USubsystem 자손 — 2026-05-29 배치 3 직후 Subsystem 카테고리 정식 신설로 자연 배치). 호스트하는 자산 [[entities/MCStoryAsset]] 은 Asset 카테고리 (배치 4 Story 직후 신설).
- `UCLASS()` `class MCPLAYMODULE_API UMCStorySubSystem : public UTickableWorldSubsystem`
- 라이프사이클: UWorldSubsystem (레벨별, World Travel 시 재생성).

## 핵심 구조
- **USubsystem Interface 오버라이드**:
  - `virtual void Initialize(FSubsystemCollectionBase&) override`
  - `virtual void Deinitialize() override`
  - `virtual void BeginDestroy() override`
- **FTickableObjectBase 오버라이드**:
  - `virtual void Tick(float DeltaTime) override`
  - `virtual TStatId GetStatId() const override`
  - `virtual bool IsTickable() const override { return true; }` — 무조건 tickable.
- **활성 Story 관리**:
  - `void RegisterStory(UMCStoryAsset*)` — 외부에서 활성 자산 지정.
  - `void UnRegisterStory()` — 해제.
  - `FGuid GetRuntimeActiveNode() const` — 현재 진행 중 노드 GUID.
- **`#if WITH_EDITOR` (Editor 측 breakpoint API)**:
  - `void OpenCurrentAsset()` — Editor 에서 현재 자산 열기.
  - `void SetRuntimeBreakPointNode(FGuid, bool Breaking)` — 런타임 breakpoint 토글.
  - `void SkipBreakPointNode(FGuid)` — breakpoint 스킵.
- **protected 상태**: `TWeakObjectPtr<UMCStoryAsset> ActivationStory` — 활성 자산 (Weak 보관 → 자산 GC 안전).

## 따르는 패턴
- 자산 호스트 패턴 — UMCStoryAsset 자체는 tickable 이 아니고, 본 Subsystem 이 RegisterStory 받아 Tick 위임 (🟢 — 헤더 API 와 STRUCTURE.md/CLAUDE.md(KMCProject) 의 명시 일치).
- Editor breakpoint 통합 — `WITH_EDITOR` 가드 안 API 만 노출. Cooked 빌드는 breakpoint API 미포함.
- TWeakObjectPtr 로 활성 자산 보관 — 자산 GC 시 nullptr 안전.

## ⚠ 함정
- **헤더 주석 부재** — 설계 결정·vault 링크가 헤더에 없음. confidence 🟡 사유.
- ~~자산 호스트 분류 충돌~~ — **해소됨** (2026-05-29 배치 3 + 4): Subsystem / Asset 카테고리 정식 신설로 자연 배치 완료.
- **PIE 다중 인스턴스** — Server PIE / Client PIE 각자 별도 World → 별도 Subsystem → 별도 ActivationStory. 글로벌 mutate 시 race 가능 (🟡 추론).
- **`IsTickable() const { return true; }` 무조건**: Activation 없는 상태에도 Tick 발화 — Tick 본체에서 `ActivationStory.IsValid()` 가드 의무.

## 연관 entity
- [[entities/MCStoryAsset]] — 활성 자산.
- [[entities/MCStory_Node]] — 노드 트리 베이스.
- [[entities/MCStory_Graph]] — 노드 컬렉션.
- Cf. [[entities/MCGameSubsystem]] — DataTable 자산 호스트 선례 (UDataTable 직접 호스트).
