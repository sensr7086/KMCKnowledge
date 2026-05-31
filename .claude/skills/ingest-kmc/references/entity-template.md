# entity 페이지 양식 (필독)

`entities/<클래스명>.md` 작성 시 이 양식을 그대로 따른다. 클래스명은 접두사 포함 실제 이름(예: `MCTableManager`, `MCCharacter`, `MCDataBase`), 파일명은 `U`/`A`/`F`/`I` 접두사를 뺀 형태로 통일한다(예: `entities/MCTableManager.md`).

## frontmatter (전 항목 필수)

```yaml
---
title: "UMCTableManager"        # 접두사 포함 정식 타입명
kind: entity
category: DataTable             # Actor | Component | DataTable (classification.md 규칙)
base_class: UObject             # 직접 상속 베이스
module: MCPLAYMODULE            # *_API 매크로 또는 .Build.cs 모듈
confidence: 🟢 VAULT            # 🟢 VAULT | 🟡 PARTIAL | 🔴 INFERRED
source_paths:                   # raw 원본 상대경로 (생성기 fallback 시 직접 열기용)
  - KMCProject/MCPlayModule/MCGame/MCTableManager.h
  - KMCProject/MCPlayModule/MCGame/MCTableManager.cpp
vault_refs:                     # 코드 주석 [[...]] 에서 추출한 mcwiki 링크
  - synthesis/subsystem-advanced-patterns
  - concepts/UE-FStructProperty-Cast-Type-Safety
last_ingested: 2026-05-29
---
```

작성 규칙:
- `vault_refs` 는 **주석에서 추출**한다. 코드에 `[[concepts/X]]` 가 있으면 그대로 옮긴다. 추론으로 추가한 링크는 본문에서 🔴 로 구분하고 frontmatter 에는 넣지 않거나 별도 표시.
- `confidence` 는 페이지 전체의 종합 신뢰도. 한 항목이라도 🔴 가 핵심이면 페이지를 🟡 이하로.
- `source_paths` 는 생성기가 1차 fallback(코드 직접 참조) 시 여는 경로. 정확해야 함.

## 본문 섹션 (순서 고정)

1. **한 줄 정의** — 이 클래스가 무엇인가, 한 문장.
2. **소속 / 상속** — 대분류, 상속 선언, UCLASS/USTRUCT 지정자(Within, BlueprintType 등), 라이프사이클.
3. **핵심 구조** — 주요 함수 시그니처, 멤버, 템플릿/매크로. BP 노출(UFUNCTION) 여부 표시.
4. **따르는 패턴** — 주석의 "설계 결정" 블록을 옮기고, 각 항목에 vault 링크.
5. **⚠ 함정** — 주석의 함정 기록. 생성기 안전장치이므로 누락 금지.
6. **연관 entity** — 같은 저장소의 다른 클래스 `[[entities/...]]`. 미인덱싱 의존은 "(미인덱싱)" 표기.

## 완성 예시 (MCTableManager — 실제 코드 기반)

```markdown
---
title: "UMCTableManager"
kind: entity
category: DataTable
base_class: UObject
module: MCPLAYMODULE
confidence: 🟢 VAULT
source_paths:
  - KMCProject/MCPlayModule/MCGame/MCTableManager.h
  - KMCProject/MCPlayModule/MCGame/MCTableManager.cpp
vault_refs:
  - synthesis/subsystem-advanced-patterns
  - concepts/Subsystem-5-Types
  - concepts/UE-FStructProperty-Cast-Type-Safety
  - concepts/UE-Phantom-Header-Hallucination-Hazard
  - entities/UGameInstance
  - concepts/Asset-Loading-Policy
last_ingested: 2026-05-29
---

# UMCTableManager

## 한 줄 정의
UDataTable 컬렉션을 Kind 단위로 관리하는 전역 관리자. UMCGameSubsystem 이 UPROPERTY 로 소유.

## 소속 / 상속
- 대분류: DataTable
- `UMCTableManager : public UObject`
- `UCLASS(Within=MCGameSubsystem, BlueprintType)` — 외부 NewObject 차단
- 라이프사이클: 부모 UMCGameSubsystem 의 Init/Deinit 가 호출

## 핵심 구조
- `Initialize(const UMCTableRegistry*)` — 모든 Kind sync TryLoad, 1회
- `Deinitialize()` — 명시적 cleanup
- `FindRowAs<T>(Kind, RowName)` — 2단 타입검사 (템플릿, 헤더 inline)
- `SafeCast<T>(const FMCDataBase*)` — static helper
- `GetTable / GetRowNames` — UFUNCTION(BlueprintCallable)
- `ReloadTable / ReloadAll` — #if WITH_EDITOR
- 멤버: Registry, LoadedTables(TMap) — UPROPERTY GC 추적

## 따르는 패턴
- 부모-자식 UObject 소유(패턴 B) → [[synthesis/subsystem-advanced-patterns]]
- 타입 검증 IsChildOf 정공법 → [[concepts/UE-FStructProperty-Cast-Type-Safety]]
- Sync 로드 1회, async Bundle 전환 여지 → [[concepts/Asset-Loading-Policy]]

## ⚠ 함정
- FindRow 반환 포인터 캐시 금지 — RowMap 재배치 dangling. 즉시 사용 후 버리기.
- phantom header — Templates/IsDerivedFrom.h 없음. TIsDerivedFrom 은 Templates/UnrealTypeTraits.h. → [[concepts/UE-Phantom-Header-Hallucination-Hazard]]
- BlueprintCallable + uint8* 반환 금지 — UHT 거부. typed 템플릿으로 우회.

## 연관 entity
- [[entities/MCDataBase]] — FindRowAs<T> 의 T 제약 베이스
- UMCGameSubsystem (소유자, 미인덱싱)
- UMCTableRegistry (Kind→경로, 미인덱싱)
```
