---
title: "UMCGameSubsystem"
kind: entity
category: Subsystem
base_class: UGameInstanceSubsystem
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCGame/MCGameSubsystem.h
  - KMCProject/MCPlayModule/MCGame/MCGameSubsystem.cpp
vault_refs:
  - entities/UGameInstance
  - concepts/Subsystem-5-Types
  - synthesis/subsystem-5-types-decision-tree
  - synthesis/subsystem-advanced-patterns
  - concepts/SeamlessTravel
last_ingested: 2026-05-29
---

# UMCGameSubsystem

## 한 줄 정의
게임 인스턴스 라이프사이클 호스트 — UMCTableManager 를 소유(패턴 B). DataTable 매니저의 외부 진입점.

## 소속 / 상속
- 대분류: DataTable (역할 기준 판정 — TableManager 호스트 + 향후 Save/Load·Online·SeamlessTravel 인계 자리)
- `UCLASS()` `class MCPLAYMODULE_API UMCGameSubsystem : public UGameInstanceSubsystem`
- 라이프사이클: 게임 실행 ~ 종료. Map 전환 살아남음(UGameInstance 호스팅).

## 핵심 구조
- **USubsystem Interface**:
  - `virtual void Initialize(FSubsystemCollectionBase&) override` — DefaultRegistryPath sync TryLoad → `StartTables(Registry)`.
  - `virtual void Deinitialize() override` — TableManager->Deinitialize() + nullptr (자식 → 자기 역순).
  - `virtual bool ShouldCreateSubsystem(UObject*) const override` — 디폴트 Super (필요 시 PLATFORM_/bDedicatedServer/UE_BUILD_SHIPPING 분기).
- **TableManager 접근**:
  - `UFUNCTION(BlueprintCallable, BlueprintPure) UMCTableManager* GetTables() const` — 외부 단일 진입점.
  - `UFUNCTION(BlueprintCallable) void SetTableRegistry(UMCTableRegistry*)` — 게임 시작 전 1회, 호출 시 기존 매니저 정리 후 재시작.
- **UPROPERTY**:
  - `EditDefaultsOnly meta=(AllowedClasses="/Script/MCPlayModule.MCTableRegistry") FSoftObjectPath DefaultRegistryPath` — 디폴트 Registry 자산 경로. 비우면 SetTableRegistry 명시 호출 대기.
  - `TObjectPtr<UMCTableManager> TableManager` — 소유 매니저. Within=MCGameSubsystem 와 페어. UPROPERTY 필수 (GC 방어).
- **private**: `void StartTables(const UMCTableRegistry*)` — `NewObject<UMCTableManager>(this)` + `Initialize(Registry)`. Outer=this → Within=MCGameSubsystem 와 페어로 외부 NewObject 차단.

## 따르는 패턴
- 합성 패턴 B (UObject 매니저 소유) → [[synthesis/subsystem-advanced-patterns]] §부모-자식 소유
- UGameInstanceSubsystem 선택 근거 (Map 전환 살아남는 데이터 위치) → [[entities/UGameInstance]] / [[concepts/Subsystem-5-Types]]
- 시나리오 #2 (세이브 매니저 호스트)·#9 (디버그 매니저) → [[synthesis/subsystem-5-types-decision-tree]]
- SeamlessTravel 인계 데이터 위치 → [[concepts/SeamlessTravel]]

## ⚠ 함정
- **PIE 다중 인스턴스 race**: Server PIE / Client PIE 각자 별도 GameInstance → 별도 TableManager. 글로벌 mutate 시 race 가능 → [[synthesis/subsystem-5-types-decision-tree]] §6.
- **Shipping 빌드 cook 불가**: MCPlayModule 의 `UnrealEd` 의존을 `if (Target.bBuildEditor)` 뒤로 옮기기 전까지 cook 실패. CLAUDE.md 명시 부채.
- Deinitialize 의 순서 강제: 자식(TableManager) → 자기. Super::Deinitialize 는 맨 마지막.

## 연관 entity
- [[entities/MCTableManager]] — 소유 매니저 (Within=MCGameSubsystem).
- [[entities/MCTableRegistry]] — DefaultRegistryPath 로 로드되는 자산.
- 향후 확장 자리: Save/Load 시스템, 글로벌 통계/업적 wrapper (Online Subsystem 래핑), SeamlessTravel 인계 데이터.
