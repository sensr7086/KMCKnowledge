---
title: "UMCLootRoller"
kind: entity
category: BlueprintLibrary
base_class: UBlueprintFunctionLibrary
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Library/MCLootRoller.h
  - KMCProject/MCPlayModule/MCLoot/Library/MCLootRoller.cpp
vault_refs:
  - concepts/CPP-BP-Boundary
  - sources/ue-coreuobject-interface
last_ingested: 2026-05-29
---

# UMCLootRoller

## 한 줄 정의
결정적 루팅 굴림 알고리즘 — 부수효과 없는 단일 static 함수 `Roll`. 향후 서버 권위 이식 용이하게 격리.

## 소속 / 상속
- 대분류: BlueprintLibrary
- `UCLASS()` `class MCPLAYMODULE_API UMCLootRoller : public UBlueprintFunctionLibrary`
- 충족 REQ: REQ-14~20. 설계 출처: `_workspace/02_ue-architect_design.md §2.3`.
- 순수 함수에 가깝게(부수효과 없음) 격리 — 향후 서버 권위 이식 용이(Q-10). (근거: `concepts/CPP-BP-Boundary "BlueprintFunctionLibrary: static UFUNCTION만, BP+C++ 양쪽"`)

## 핵심 구조
- **`static void Roll(...)`** `UFUNCTION(BlueprintCallable, Category="MC|Loot", meta=(WorldContext="WorldContextObject"))`:
  - 인자: `const UObject* WorldContextObject` (TableManager 접근용), `const UMCLootTableAsset* LootTable`, `int32 Seed` (0=비결정 GenerateNewSeed, 그 외=결정적 재현, REQ-15), `UMCLootableComponent* ConditionSource` (condition 평가 위임 대상; null이면 빈 조건만 통과), `TArray<FMCLootResultItem>& OutResults` (결과 누적, REQ-20).
- **TableManager 단일 진입 (CP-1 Option B)**: `GEngine->GetWorldFromContextObject` → `World->GetGameInstance()` → `GetSubsystem<UMCGameSubsystem>()->GetTables()`. 실패 시 Warning + 빈 결과.
- **결정적 RNG**: `FRandomStream Stream`; `UsedSeed = (Seed != 0) ? Seed : FMath::Rand()`; `Stream.Initialize(UsedSeed)`. (REQ-15)
- **풀 스냅샷**: `EntryRowNames` 의 각 RowName 을 `Tables->FindRowAs<FMCData_LootEntry>(EntryTableKind, RowName)` 로 가져와 `Entries.Add(*Row)` **즉시 값 복사**(dangling 회피). 미해결 row 는 Warning(CP-1 이중 동기화 확인 메시지) 후 skip.
- **조건 평가 (파일 로컬 익명 namespace)**: `bool ConditionOk(const FMCData_LootEntry&, UMCLootableComponent*)` — `Condition.IsNone()`이거나 빈 문자열이면 true; ConditionSource null 이면 보수적으로 false(잘못된 무조건 지급 방지); 아니면 `UMCLootableComponent::Execute_IsConditionMet(ConditionSource, Condition)`. (근거: `sources/ue-coreuobject-interface §5 Execute_*` — BlueprintNativeEvent 는 Execute_ 래퍼로 호출)
- 로그 카테고리: `DEFINE_LOG_CATEGORY_STATIC(LogMCLoot, Log, All)`.

### 알고리즘 (PDF §5)
1. **보장 단계** (REQ-12, REQ-16): `bIsGuaranteed && ConditionOk` 항목 → weight 무시, `Stream.RandRange(Min,Max)` 수량 무조건 추가(Count>0 일 때).
2. **Rolls 회 반복** (REQ-17~19):
   - `Stream.FRand() < EmptyChance` 면 skip(꽝). (REQ-17)
   - 가중치 풀 = `!bIsGuaranteed && ConditionOk`, `Total = Σ max(0, Weight)`. (REQ-18)
   - `Total <= 0` 이면 skip. (REQ-19)
   - `Pick = RandRange(0, Total-1)`, 누적합으로 1개 선택 → `RandRange(Min,Max)` 수량 추가. (REQ-18, REQ-20)
- **디버그 로깅**(`#if !UE_BUILD_SHIPPING`): TableId / UsedSeed / Rolls / EmptyChance / 결과 목록. (REQ-15, T-13)

## 따르는 패턴
- BlueprintFunctionLibrary = static UFUNCTION 만, BP+C++ 양쪽 → [[concepts/CPP-BP-Boundary]]
- BlueprintNativeEvent 호출은 `Execute_*` 래퍼 경유 → [[sources/ue-coreuobject-interface]] §5
- 부수효과 없는 격리(서버 권위 이식 용이, Q-10) — `OutResults.Reset()` 후 순수 입력→출력.
- 값/멤버십 분리(CP-1 Option B): `EntryRowNames` 의 RowName 만 공용 DT 에서 로드 → [[entities/MCLootTableAsset]] / [[entities/MCData_LootEntry]]

## ⚠ 함정
- **FRandomStream 근거 미확보 (🟡)**: 코드 주석이 명시 — "MCWiki FRandomStream 페이지 없음". Core 표준 API(`Core/Public/Math/RandomStream.h`): `FRandomStream(int32)`, `FRand()`, `RandRange(Min,Max) inclusive`, `GenerateNewSeed()`. MCWiki list_pages(sources/concepts) 확인 결과 실제로 해당 페이지 없음 — 코드의 자기-신고가 정확함. 본 항목은 엔진 표준 API 지식으로 메운 것이므로 🟡.
- **FindRow 반환 포인터 dangling**: RowMap 재배치 시 dangling → 즉시 값 복사(`Entries.Add(*Row)`)로 우회. 베이스 [[entities/MCTableManager]] 함정. (코드 주석 명시)
- **TableManager 접근 실패 = 빈 결과**: WorldContext/Subsystem 해석 실패 또는 Tables null 이면 Warning 후 빈 OutResults 반환(silent crash 회피).
- **CP-1 이중 동기화 누락 탐지**: `EntryRowNames` 의 RowName 이 공용 DT 에 없으면 Warning 로그("DT 와 EntryRowNames 동기화 확인 — CP-1 이중 동기화 비용") 후 해당 항목 skip → 풀 축소/빈 풀 가능. → [[entities/MCLootTableAsset]] CP-1.
- ConditionSource null + 비어있지 않은 Condition = 보수적 미통과(false). 잘못된 무조건 지급 방지 의도.

## 연관 entity
- [[entities/MCLootTableAsset]] — 굴릴 테이블 자산(`EntryRowNames`/`EntryTableKind`/`Rolls`/`EmptyChance`).
- [[entities/MCData_LootEntry]] — 풀을 구성하는 row 값 타입 + `FMCLootResultItem` 생산.
- [[entities/MCTableManager]] — `FindRowAs<FMCData_LootEntry>` 로드 경로 (GetTables 경유).
- [[entities/MCGameSubsystem]] — `GetTables()` 진입점 (WorldContext → GameInstance).
- [[entities/MCLootableComponent]] — `Execute_IsConditionMet` 위임 대상이자 `Roll` 의 주 호출처.
