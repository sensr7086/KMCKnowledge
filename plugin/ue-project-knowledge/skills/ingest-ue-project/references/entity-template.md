# entity 페이지 양식 (필독)

`entities/<클래스명>.md` 작성 시 이 양식을 그대로 따른다. 클래스명은 접두사 포함 실제 이름(예: `MyTableManager`, `MyCharacter`, `MyDataBase`), 파일명은 `U`/`A`/`F`/`I` 접두사를 뺀 형태로 통일한다(예: `entities/MyTableManager.md`).

## frontmatter (전 항목 필수)

```yaml
---
title: "UMyTableManager"        # 접두사 포함 정식 타입명
kind: entity
category: DataTable             # categories.md 의 활성 카테고리 중 하나
base_class: UObject             # 직접 상속 베이스
module: MYPROJECTMODULE         # *_API 매크로 또는 .Build.cs 모듈명
confidence: 🟢 VAULT            # 🟢 VAULT | 🟡 PARTIAL | 🔴 INFERRED
source_paths:                   # raw 원본 상대경로 (생성기 fallback 시 직접 열기용)
  - MyProject/MyModule/MyGame/MyTableManager.h
  - MyProject/MyModule/MyGame/MyTableManager.cpp
vault_refs:                     # 코드 주석 [[...]] 에서 추출한 mcwiki 링크
  - synthesis/subsystem-advanced-patterns
  - concepts/UE-FStructProperty-Cast-Type-Safety
last_ingested: 2026-05-29
---
```

작성 규칙:

- `vault_refs` 는 **주석에서 추출**한다. 코드에 `[[concepts/X]]` 가 있으면 그대로 옮긴다. 추론으로 추가한 링크는 본문에서 🔴 로 구분하고 frontmatter 에는 넣지 않거나 별도 표시.
- `confidence` 는 페이지 전체의 종합 신뢰도. 한 항목이라도 🔴 가 핵심이면 페이지를 🟡 이하로.
- `source_paths` 는 생성기가 1차 fallback(코드 직접 참조) 시 여는 경로. 정확해야 함. 인스턴스의 `CODE_REPO_PATH` 기준 상대경로.
- `category` 는 categories.md 의 활성 카테고리 중 하나여야 함. 예약 카테고리 사용 시 신설 절차 4단계 수반.

## 본문 섹션 (순서 고정)

1. **한 줄 정의** — 이 클래스가 무엇인가, 한 문장.
2. **소속 / 상속** — 카테고리, 상속 선언, UCLASS/USTRUCT 지정자(Within, BlueprintType 등), 라이프사이클. 분류 한계가 있으면 여기에 명시.
3. **핵심 구조** — 주요 함수 시그니처, 멤버, 템플릿/매크로. BP 노출(UFUNCTION) 여부 표시.
4. **따르는 패턴** — 주석의 "설계 결정" 블록을 옮기고, 각 항목에 vault 링크.
5. **⚠ 함정** — 주석의 함정 기록. 생성기 안전장치이므로 누락 금지.
6. **연관 entity** — 같은 저장소의 다른 클래스 `[[entities/...]]`. 미인덱싱 의존은 "(미인덱싱 — <카테고리> 배치)" 표기.

## 완성 예시 (UMyTableManager — DataTable 카테고리, generic UE)

```markdown
---
title: "UMyTableManager"
kind: entity
category: DataTable
base_class: UObject
module: MYPROJECTMODULE
confidence: 🟢 VAULT
source_paths:
  - MyProject/MyModule/MyGame/MyTableManager.h
  - MyProject/MyModule/MyGame/MyTableManager.cpp
vault_refs:
  - synthesis/subsystem-advanced-patterns
  - concepts/Subsystem-5-Types
  - concepts/UE-FStructProperty-Cast-Type-Safety
  - concepts/UE-Phantom-Header-Hallucination-Hazard
  - entities/UGameInstance
  - concepts/Asset-Loading-Policy
last_ingested: 2026-05-29
---

# UMyTableManager

## 한 줄 정의
UDataTable 컬렉션을 Kind 단위로 관리하는 전역 관리자. UMyGameSubsystem 이 UPROPERTY 로 소유.

## 소속 / 상속
- 카테고리: DataTable (역할 기준 — UDataTable 컬렉션 관리)
- `UMyTableManager : public UObject`
- `UCLASS(Within=MyGameSubsystem, BlueprintType)` — 외부 NewObject 차단
- 라이프사이클: 부모 UMyGameSubsystem 의 Init/Deinit 가 호출

## 핵심 구조
- `Initialize(const UMyTableRegistry*)` — 모든 Kind sync TryLoad, 1회
- `Deinitialize()` — 명시적 cleanup
- `FindRowAs<T>(Kind, RowName)` — 2단 타입검사 (템플릿, 헤더 inline)
- `SafeCast<T>(const FMyDataBase*)` — static helper
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
- [[entities/MyDataBase]] — FindRowAs<T> 의 T 제약 베이스
- UMyGameSubsystem (소유자, 미인덱싱 — Subsystem 배치)
- UMyTableRegistry (Kind→경로, 미인덱싱 — DataTable 배치)
```

## 카테고리별 본문 강조점 (참고)

- **Actor / Component**: 라이프사이클(BeginPlay/Tick/EndPlay) override 순서, UPROPERTY GC 안전, BP 노출 비트.
- **Subsystem**: 라이프사이클 범위(Game/World/Tickable/Editor), `ShouldCreateSubsystem` 조건, 합성 패턴.
- **Interface**: UINTERFACE specifier(MinimalAPI/Blueprintable/BlueprintType), BlueprintNativeEvent 패턴, default 구현.
- **DataTable**: 행 다형성(IsA<T>/CastTo<T>), 매크로(MC_DATA_BODY 등), RowMap dangling.
- **AssetUserData**: UCLASS specifier(DefaultToInstanced/EditInlineNew/CollapseCategories), Draw/PostEditChangeOwner/PostEditUndo, BlueprintNativeEvent 미사용.
- **Asset (예약)**: 자산 콘텐츠(그래프 / 데이터 묶음), Cooking, Soft 참조 정책.
- **GraphNode (예약)**: 부모-자식 GUID 토폴로지, Abstract+Blueprintable, 가상 hook (OnBegin/Tick/End).
- **BlueprintLibrary (예약)**: 모든 함수 static + UFUNCTION(BlueprintCallable), 외부 모듈 의존 최소화.
- **Editor (예약)**: `#if WITH_EDITOR` 가드, MCEditorModule 모듈, Slate 라이프사이클(Construct/OnPaint).
