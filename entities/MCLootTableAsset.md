---
title: "UMCLootTableAsset"
kind: entity
category: Asset
base_class: UDataAsset
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCLoot/Data/MCLootTableAsset.h
vault_refs:
  - sources/ue-assetclasses-data
last_ingested: 2026-05-29
---

# UMCLootTableAsset

## 한 줄 정의
단일 루팅 테이블 정의 — UDataAsset. table_id / rolls / empty_chance 스칼라 메타 + 공용 LootEntry DT row 의 큐레이션 부분집합(`EntryRowNames`)을 보유.

## 소속 / 상속
- 대분류: Asset
- `UCLASS(BlueprintType)` `class MCPLAYMODULE_API UMCLootTableAsset : public UDataAsset`
- 충족 REQ: REQ-8, REQ-9. 설계 출처: `_workspace/02_ue-architect_design.md §2.2`.
- 동일 UDataAsset 패턴: [[entities/MCTableRegistry]] 가 동일 사용. (근거: 헤더 주석 — `sources/ue-assetclasses-data §UDataAsset "단순 데이터 컨테이너 BP/C++ 양쪽"`)

## 핵심 구조
- **UPROPERTY (EditAnywhere, BlueprintReadWrite, Category="MC|Loot")**:
  - `FName TableId` — 로깅/디버그 식별자. (REQ-9)
  - `int32 Rolls = 1` `meta=(ClampMin="0")` — 가중치 굴림 반복 횟수. (REQ-9)
  - `float EmptyChance = 0.f` `meta=(ClampMin="0.0", ClampMax="1.0")` — 매 굴림 시작 시 꽝(skip) 확률 [0,1]. (REQ-9)
  - `EMCTableKind EntryTableKind = EMCTableKind::LootEntry` — 엔트리 값을 조회할 공용 DataTable 종류(TableManager 경유 키). (CP-1 Option B)
  - `TArray<FName> EntryRowNames` — ★ CP-1 핵심 필드. 이 풀에 속하는 row 의 멤버십(큐레이션 부분집합). 공용 LootEntry DataTable 에서 이 테이블이 의도적으로 선택한 RowName 목록. (REQ-9)
- DT *값* 은 보유하지 않음 — 값의 단일 출처는 공용 LootEntry DT. [[entities/MCLootRoller]] 가 `EntryRowNames` 의 RowName 만 `FindRowAs<FMCData_LootEntry>` 로 가져와 풀 구성.

## 따르는 패턴
- 경로/로드 책임은 전적으로 TableManager — 이 자산은 `TSoftObjectPtr<UDataTable>` 를 직접 보유하지 **않음**. → [[sources/ue-assetclasses-data]] §UDataAsset
- 스칼라 메타(table_id/rolls/empty_chance)는 자산이 직접 보유, entries 는 공용 DT row 부분집합으로 위임. (헤더 주석 PDF §3 LootTable 매핑)

### ★ CP-1 결정 (Option B) — future-run 안전장치
헤더 클래스 주석과 `EntryRowNames` 필드 주석에 명시된 사용자 결정사항을 그대로 인용·정리:

- **결정**: `EntryRowNames(TArray<FName>)` 로 공용 LootEntry DT 의 row **멤버십**을 테이블별 큐레이션 부분집합으로 보유 = **의도된 설계(Option B)**.
- **이유**: 엔트리 값은 공용 DT 에 두되, "이 루팅 풀에 어떤 row 가 속하는가"는 풀마다 다르므로 하나의 공용 DT 에서 풀별로 의도적으로 큐레이션한 부분집합을 골라야 하는 요구가 있음.
- **row 데이터 접근**: TableManager `FindRowAs<T>` 단일 경로 유지 — 자산에 `TSoftObjectPtr<UDataTable>` 직접 보유 **없음**(DT 경로 이중화 회피). 헤더 주석: "DT 경로 이중화는 과거 포스트모템이 STOP 처리한 안티패턴".
- **비용 (의도적 수용)**: 공용 LootEntry DT 에 row 추가/이름변경/삭제 시 그 row 를 참조하는 풀의 `EntryRowNames` 도 함께 갱신해야 하는 **이중 동기화 비용** 발생. 직전 사용검증(CP-1)에서 지적된 항목을 사용자 결정(Option B)으로 정당화함.

> 근거: 본 결정은 추론이 아니라 코드 헤더 주석(클래스 블록 + `EntryRowNames` 필드 블록)에 실제로 기록되어 있음 — 🟢.

## ⚠ 함정
- **EntryRowNames 이중 동기화 (CP-1)**: 위 CP-1 결정의 비용. 공용 LootEntry DT 에서 row 를 추가/이름변경/삭제하면 이 목록도 함께 갱신해야 함. 미해결 row 는 [[entities/MCLootRoller]] 가 Warning 로그 후 skip(빈 풀 가능).
- **DT 경로 이중화 금지**: 이 자산에 `TSoftObjectPtr<UDataTable>` 를 추가하지 말 것 — 과거 포스트모템 STOP 처리 안티패턴. 경로/로드는 TableManager 책임. (헤더 주석 명시)
- **EntryTableKind 매핑 의무**: `EMCTableKind::LootEntry` 가 Registry 자산(.uasset)의 TablePaths 에 등록돼 있어야 `FindRowAs` 가 동작 — [[entities/MCTableRegistry]] 의 사용자 수동 매핑 의존. (미등록 시 Roller 가 Warning + 빈 결과)

## 연관 entity
- [[entities/MCData_LootEntry]] — `EntryRowNames` 가 가리키는 공용 DT row 의 값 타입.
- [[entities/MCTableManager]] — `FindRowAs<FMCData_LootEntry>(EntryTableKind, RowName)` 단일 로드 경로.
- [[entities/MCTableRegistry]] — `EMCTableKind::LootEntry` Kind → DT 경로 매핑 자산.
- [[entities/MCLootRoller]] — 본 자산을 굴리는 라이브러리.
- [[entities/MCLootableComponent]] — 본 자산을 `LootTable` 로 참조하는 런타임 소비자.
