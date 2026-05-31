# KMCKnowledge 저장소 구축 사양 (STRUCTURE)

> 이 문서는 Cowork 에이전트가 **KMCKnowledge 저장소를 처음 구축**할 때 따르는 일회성 사양서다.
> 저장소가 만들어진 뒤의 **운영·작업 규약**은 `CLAUDE.md` 가 담는다. 두 문서를 같이 둔다.

---

## 0. 사전 정보 (작업 전 사용자 확인)

다음 두 경로를 사용자에게 확인하고 진행한다. 임의 추정 금지.

- **raw 출처**: KMCProject 코드 루트 (기본 후보 `E:\MCProject\KMCProject`)
- **저장소 위치**: KMCProject **밖**의 별도 디렉토리 (기본 후보 `E:\MCProject\KMCKnowledge`)

저장소는 KMCProject 안에 두지 않는다. 코드 저장소와 지식 저장소는 독립적으로 버전 관리·갱신된다.

---

## 1. 디렉토리 구조 (생성 직후 상태)

다음 트리를 그대로 생성한다. `<...>` 표기는 placeholder, 실제로는 생성하지 않는다.

```
KMCKnowledge/
├── CLAUDE.md                 # 운영 헌법 — 이 저장소에서 작업하는 모든 에이전트가 먼저 읽음
├── STRUCTURE.md              # 본 문서 — 구축 사양 (초기 구축 후 보존)
├── categories.md             # 카테고리 레지스트리 — 활성/예약/폐기 + 자동 분류 규칙 (SSOT)
├── index.md                  # 전체 카탈로그 · 생성기 locate 진입점 · 매 ingest 후 재생성
├── catalogs/                 # 카테고리당 1 파일. 신설은 categories.md 의 절차로.
│   ├── actor.md              # Actor 대분류 목차
│   ├── component.md          # Component 대분류 목차
│   ├── datatable.md          # DataTable 대분류 목차
│   ├── subsystem.md          # Subsystem 대분류 (USubsystem 자손)
│   ├── interface.md          # Interface 대분류 (I*Interface)
│   └── assetuserdata.md      # AssetUserData 대분류 (UAssetUserData 자손)
├── entities/                 # 클래스별 정제 페이지 — 초기엔 비어 있어도 됨
│   └── .gitkeep              # 빈 디렉토리 유지용
├── .claude/                  # Claude Code / Cowork 자동 발견 위치
│   └── skills/
│       └── ingest-kmc/       # 인덱서 스킬 (SKILL.md + references/)
└── raw/                      # KMCProject 코드 사본 또는 심볼릭 참조 — 운영 정책은 §5
    └── .gitkeep
```

> **주의 — 본 §1 은 *현 시점* 의 트리**. 초기 구축 직후에는 3 카탈로그(actor/component/datatable)만 존재했다. Subsystem/Interface/AssetUserData 카탈로그와 categories.md, .claude/skills/ 는 운영 중 신설된 항목으로, 본 문서가 보존 사양(snapshot)이라는 원칙에 따라 트리에 반영해 둔다. 향후 카테고리 신설은 [[categories]] 의 신설 절차를 따른다.

---

## 2. 명명 규약 (전 파일 공통)

- **entity 파일명**: 타입 접두사(`U`/`A`/`F`/`I`)를 제거한 형태. 예) `UMCTableManager` → `MCTableManager.md`, `FMCDataBase` → `MCDataBase.md`, `AMCCharacter` → `MCCharacter.md`
- **frontmatter `title` 필드**: 접두사 포함 정식 타입명 유지. 파일명과 다름.
- **catalog 파일명**: 대분류 소문자, separator 없음 (`actor.md`, `component.md`, `datatable.md`, `subsystem.md`, `interface.md`, `assetuserdata.md`). 신설 카테고리는 [[categories]] §명명 규약 준수.
- **내부 링크**: `[[entities/MCTableManager]]`, `[[catalogs/datatable]]` 형식. 파일명 기준(접두사 없음).
- **신뢰도 태그**: 🟢 VAULT / 🟡 PARTIAL / 🔴 INFERRED. frontmatter `confidence` 와 본문 인라인 표기 양쪽에 사용.

---

## 3. 초기 파일 내용

### 3.1 `index.md` (초기 상태)

ingest 가 한 번도 안 돌았어도 다음 골격은 갖춰 둔다. ingest 가 돌면 카운트·목록·날짜를 자동 갱신한다.

```markdown
# KMCKnowledge Index

> Last ingested: (없음 — 초기 상태) · **0 entities / 3 catalogs** · raw = (사용자 확인 경로)
> 매 ingest 후 자동 갱신. 생성기 locate 단계의 진입점.

---

## 저장소 역할
- 본 저장소 = **KMCProject 전용 지식** (이 프로젝트는 *실제로* 이렇게 짜여 있다)
- UE 일반 지식·함정은 **mcwiki** (MCP) 참조 — vault_refs 로 링크

## 신뢰도 범례
- 🟢 VAULT — ingest 로 정제·검증된 스킬
- 🟡 PARTIAL — 코드 직접 추론 (정제 전)
- 🔴 INFERRED — 엔진 지식(mcwiki)으로 구성, 프로젝트 선례 없음

---

## 대분류 (catalogs)

### Actor
- [[catalogs/actor]]

### Component
- [[catalogs/component]]

### DataTable
- [[catalogs/datatable]]

---

## 미인덱싱 큐
(초기에는 비어 있음. ingest 시 raw 에 있으나 미정제인 클래스를 여기로 옮긴다.)
```

### 3.2 `catalogs/<대분류>.md` (초기 상태)

세 파일 모두 같은 골격. `<대분류>` 자리는 각 파일에 맞게.

```markdown
---
title: "<대분류> 대분류"
kind: catalog
category: <Actor|Component|DataTable>
last_ingested: (없음)
---

# <대분류> 대분류

> KMCProject 의 <대분류> 관련 구현체. 베이스 → 관리자 → 구현체 순으로 정리.

## 베이스
(ingest 시 채움)

## 관리자
(ingest 시 채움)

## 구현체
(ingest 시 채움)

## 공통 함정
(ingest 시 entity 들의 공통 함정을 모음)
```

### 3.3 `entities/` (초기 상태)

빈 디렉토리 + `.gitkeep`. 첫 ingest 가 페이지를 채운다.

### 3.4 `raw/` 운영 정책

raw 는 KMCProject 코드를 **출처로 참조**하는 자리다. 다음 중 한 방식으로 운영하며, 사용자가 선택한다.

- (A) **참조만**: raw/ 는 빈 디렉토리로 두고, 모든 코드 읽기는 KMCProject 실제 경로에서 한다.
  → entity frontmatter 의 `source_paths` 는 KMCProject 루트 기준 상대경로.
- (B) **사본**: 인덱싱 대상 코드를 raw/ 로 복사하여 스냅샷 보관.
  → 갱신 시점이 명확하지만 동기화 책임이 생긴다.
- (C) **심볼릭 링크**: raw/ 가 KMCProject 디렉토리를 가리키는 symlink (Windows: junction).
  → 항상 최신이지만 OS·도구 의존성 있음.

초기 권장: **(A) 참조만**. 단순하고 동기화 이슈 없음.

---

## 4. entity 페이지 양식 (ingest 가 채울 때 따르는 규칙)

이 섹션은 **초기 구축 단계에선 만들 일이 없지만**, STRUCTURE.md 에 명시해 두어 이후 ingest 작업의 단일 진실원(SSOT)으로 삼는다.

### 4.1 frontmatter (전 항목 필수)

```yaml
---
title: "UMCTableManager"        # 접두사 포함 정식 타입명
kind: entity
category: DataTable             # Actor | Component | DataTable
base_class: UObject             # 직접 상속 베이스
module: MCPLAYMODULE            # *_API 매크로 또는 .Build.cs 모듈명
confidence: 🟢 VAULT            # 🟢 VAULT | 🟡 PARTIAL | 🔴 INFERRED
source_paths:                   # raw 출처 (생성기 fallback 시 직접 열기용)
  - KMCProject/MCPlayModule/MCGame/MCTableManager.h
  - KMCProject/MCPlayModule/MCGame/MCTableManager.cpp
vault_refs:                     # 코드 주석 [[...]] 에서 추출한 mcwiki 링크
  - synthesis/subsystem-advanced-patterns
  - concepts/UE-FStructProperty-Cast-Type-Safety
last_ingested: 2026-05-29
---
```

### 4.2 본문 섹션 (순서 고정)

1. **한 줄 정의** — 이 클래스가 무엇인가, 한 문장.
2. **소속 / 상속** — 대분류, 상속 선언, UCLASS/USTRUCT 지정자, 라이프사이클.
3. **핵심 구조** — 주요 함수 시그니처, 멤버, 템플릿/매크로. BP 노출 여부.
4. **따르는 패턴** — 주석의 "설계 결정" 블록 + vault 링크.
5. **⚠ 함정** — 주석의 함정 기록. 생성기 안전장치.
6. **연관 entity** — 다른 entity 링크, 미인덱싱 의존은 "(미인덱싱)" 표기.

### 4.3 작성 원칙

- **추론보다 추출**: 코드 주석에 `[[...]]` 가 있으면 그대로 옮긴다. 새로 추론한 항목은 본문에서 🔴 표기.
- **빈칸을 지어내지 않는다**: 근거 없으면 confidence 를 🟡/🔴 로 낮추고 누락 사실을 본문에 명시.
- **mcwiki 는 읽기만**: vault_refs 검증을 위한 `read_page` 호출 외에 mcwiki 를 수정하지 않는다.

---

## 5. 대분류 판정 규칙 (ingest 시 사용)

> **단일 진실원** = [[categories]]. 본 §5 는 빠른 참조용 요약. 신설·자동 분류 규칙의 권위 정의는 categories.md.

| 상속/특징 | 대분류 |
|---|---|
| `A...` (ACharacter / AActor / APawn / ACameraActor / APhysicsVolume 등) + Actor 보조 UObject (Camera Mode 등) | **Actor** |
| `U...Component` (UActorComponent / U*MeshComponent 등 자손) + `UAnimInstance` 자손 | **Component** |
| `USubsystem` 자손 (UGameInstance / UWorld / UTickableWorld / UEditor / UEngine / ULocalPlayerSubsystem) | **Subsystem** |
| `I...Interface` (UINTERFACE 또는 plain C++) | **Interface** |
| `FTableRowBase` 자손 + `UDataTable` 로드·관리 역할 UObject | **DataTable** |
| `UAssetUserData` 자손 | **AssetUserData** |

**예약 카테고리** (트리거 시 자동 신설):
- **Asset** — UMCStoryAsset / UMCPartsAsset / UDataAsset / UPrimaryDataAsset (DataTable 자손 제외)
- **GraphNode** — UMCStory_* / UMCParts_* / UMCCombo* 그래프 노드 자손
- **BlueprintLibrary** — UBlueprintFunctionLibrary 자손
- **Editor** — MCEditorModule 의 F*/S*/UFactory/UEdGraphNode/UEdGraphSchema

분류 매칭 실패 시 [[categories]] §신설 절차 1번 (사용자 보고) 발화. 기존 엣지 케이스 — 인터페이스/에셋 — 는 본 카테고리화로 해소됨. CLAUDE.md §엣지 케이스의 잔존 항목 (다단 상속 USTRUCT / 미해결 의존 / 분류 불가) 만 별도 처리.

---

## 6. 구축 절차 (Cowork 에이전트가 따를 순서)

1. 사용자에게 raw 출처·저장소 위치 확인. 임의 추정 금지.
2. 위 §1 트리대로 디렉토리·파일 생성.
3. `index.md`, `catalogs/*.md` 를 §3 의 초기 골격으로 채움.
4. `entities/.gitkeep`, `raw/.gitkeep` 생성.
5. `CLAUDE.md` 를 저장소 루트에 함께 배치 (별도 파일로 제공됨).
6. raw 운영 방식 (A/B/C) 을 사용자에게 확인.
7. 구축 완료 보고 — 다음 단계는 `/ingest` 스킬로 첫 인덱싱.

---

## 7. 두 워크플로우의 위치 (참고)

이 저장소는 두 워크플로우의 접점이다. 운영 시 혼동 방지를 위해 명시한다.

- **워크플로우 1 — 인덱서** (`/ingest` 스킬): KMCProject 코드를 읽어 이 저장소에 **쓴다**.
- **워크플로우 2 — 생성기** (analyze-spec → locate → plan → implement): 기획서를 받아 이 저장소를 **읽고** KMCProject 에 코드 파일을 쓴다.

본 STRUCTURE.md 는 워크플로우 1·2 가 모두 동작할 수 있는 **그릇(저장소 구조)** 까지만 책임진다. 워크플로우 자체의 동작 규약은 각 스킬의 SKILL.md 와 본 저장소의 CLAUDE.md 가 정의한다.
