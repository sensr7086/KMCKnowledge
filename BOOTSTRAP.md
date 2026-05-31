# BOOTSTRAP.md — 외부 지식 저장소 생성 키트 (`ue-project-knowledge`)

> **이 파일 하나로** 빈 폴더에서 새 UE 프로젝트 전용 지식 저장소를 세운다.
> KMCKnowledge(첫 인스턴스)와 동일한 구조 — CLAUDE.md / STRUCTURE.md / project.config.md / categories.md / index.md + catalogs/ + entities/ — 를 **자가 복제**한다.
> 원본 KMCKnowledge 에 접근할 필요가 없다. 필요한 모든 템플릿이 이 문서 안에 박혀 있다.

---

## 0. 에이전트 지시 — 이 파일을 인식하면 (READ-AND-EXECUTE)

**이 파일을 읽은 에이전트는 즉시 아래 §2 「부트스트랩 프로토콜」을 실행한다.**

다만 본 키트는 원본 저장소의 사람-인-더-루프 철학(CLAUDE.md §2.6 / §3 / §6 체크포인트)을 그대로 계승한다. 따라서 "자동 진행"은 다음을 의미한다.

- 인스턴스 변수는 **AskUserQuestion 으로 한 번에 받는다** (추측하지 않는다).
- 변수 확정 → 디렉토리 골격 → 5개 핵심 문서 생성까지는 **연속 자동 수행**한다.
- **ingest(코드 인덱싱) 직전에 반드시 멈춘다.** 첫 인덱싱 진행 여부는 사용자가 결정한다.
- 대상 폴더에 기존 파일이 있으면 **덮어쓰기 전에 보고하고 확인을 받는다.**

> 트리거 예시: 사용자가 빈 폴더를 열고 "BOOTSTRAP 읽어줘" / "이 폴더에 지식저장소 만들어줘" / "ue-project-knowledge 새 인스턴스 세워줘" 라고 말하면 이 절차가 발화된다.

---

## 1. 이 키트가 만드는 것

```
<새-저장소>/
├── BOOTSTRAP.md       # (이 파일) 다음 인스턴스를 위해 남겨둠 — 선택
├── CLAUDE.md          # 운영 헌법 (항상 가장 먼저 읽음)
├── STRUCTURE.md       # 초기 구축 사양 / 양식 정식 정의
├── project.config.md  # 인스턴스 변수 단일 진실원
├── categories.md      # 대분류 권위 정의
├── index.md           # 저장소 진입점 (생성기 locate 시작점)
├── catalogs/          # 대분류별 entity 목록 (빈 골격)
└── entities/          # 클래스별 상세 페이지 (.gitkeep 만)
```

산출물은 **골격 + 5개 핵심 문서**다. entity 페이지는 이후 ingest 워크플로우가 채운다.

---

## 2. 부트스트랩 프로토콜 (에이전트 실행 절차)

### Step 0 — 대상 폴더 확인
- 현재 작업 폴더(사용자가 연 폴더)를 새 저장소 루트로 삼는다.
- 폴더에 기존 `CLAUDE.md` 등이 있으면 **즉시 멈추고** 사용자에게 보고: 덮어쓸지 / 다른 폴더를 쓸지.

### Step 1 — 인스턴스 변수 수집 (필수, AskUserQuestion 1회)
§3 표의 변수를 한 번에 묻는다. 사용자가 답하지 않은 항목은 §3 의 기본값을 쓰되, **추측한 값임을 명시**한다. `{{RAW_ROOT}}` 는 절대경로를 박지 말고 사용자 환경 기준으로 받는다.

### Step 2 — 디렉토리 골격 생성
`catalogs/`, `entities/` 를 만들고 `entities/.gitkeep` 을 둔다.

### Step 3 — 5개 핵심 문서 생성 (변수 치환)
§5~§9 템플릿을 그대로 쓰되 `{{...}}` 자리표시자를 Step 1 값으로 치환해 각 파일을 쓴다. 자리표시자를 **하나라도 남기지 않는다** (CLAUDE.md §2.5 빈칸 금지 계승).

### Step 4 — catalogs 빈 골격 + index 초기화
활성 카테고리마다 `catalogs/<소문자>.md` 를 entity 0 상태로 만들고, `index.md` 를 통계 0 으로 초기화한다(§9).

### Step 5 — 체크포인트 (여기서 멈춤)
다음을 보고하고 사용자 결정을 기다린다.
- 생성된 파일 목록
- 채워야 할 `(unknown)` / 추측 변수 항목
- "지금 첫 ingest 를 진행할까요? (raw 경로: `{{RAW_ROOT}}`)" — **여기서 자동 진행을 멈춘다.**

> 첫 ingest 는 `/ingest` 스킬(또는 동등 워크플로우)이 따로 담당한다. 본 키트는 ingest 를 실행하지 않는다.

---

## 3. 인스턴스 변수 (Step 1 에서 수집)

| 키 | 의미 | 예시 (KMC 인스턴스) | 기본값 |
|---|---|---|---|
| `{{PROJECT_NAME}}` | 대상 UE 게임 프로젝트명 | KMCProject | (필수) |
| `{{VAULT_NAME}}` | 이 지식 저장소 이름 | KMCKnowledge | `{{PROJECT_NAME}}Knowledge` |
| `{{PLUGIN_NAME}}` | 이 패턴의 플러그인명 | ue-project-knowledge | `ue-project-knowledge` |
| `{{UE_VERSION}}` | 대상 언리얼 엔진 버전 | 5.7.4 | (필수) |
| `{{MCWIKI_SERVER}}` | UE 일반지식 mcwiki MCP 서버명 | MCWiki - UE 5.7.4 Knowledge Vault | `MCWiki - UE {{UE_VERSION}} Knowledge Vault` |
| `{{RAW_ROOT}}` | raw 코드(대상 프로젝트) 실제 경로 | (사용자 환경) | `(unknown)` |
| `{{RAW_SOURCE_GLOB}}` | 인덱싱 대상 소스 glob | `Source/**/*.{h,cpp}` | `Source/**/*.{h,cpp}` |
| `{{CLASS_PREFIX}}` | 프로젝트 클래스 접두사 (분류 트리거용) | MC | (선택, 없으면 비움) |
| `{{TODAY}}` | 오늘 날짜 (YYYY-MM-DD) | 2026-05-29 | (실행일) |

> `{{MCWIKI_SERVER}}` 가 실제 연결돼 있지 않아도 부트스트랩은 진행된다. mcwiki 는 읽기 전용 보편지식 소스로서 ingest/locate 단계에서만 필요하다(CLAUDE.md §2.1).

---

## 4. 디렉토리 골격 (Step 2)

```
<루트>/
├── catalogs/
└── entities/
    └── .gitkeep
```

활성 카테고리 6개 → `catalogs/actor.md`, `component.md`, `subsystem.md`, `interface.md`, `datatable.md`, `assetuserdata.md` (Step 4 에서 빈 골격 생성).

---

## 5. 템플릿 — `CLAUDE.md` (운영 헌법)

> 아래 4-백틱 블록 내용을 `{{...}}` 치환 후 그대로 `CLAUDE.md` 로 쓴다.

````markdown
# CLAUDE.md — {{VAULT_NAME}} 운영 헌법

> 이 저장소({{VAULT_NAME}})에서 작업을 시작하는 모든 에이전트는 **이 문서를 가장 먼저 읽고** 그 규약 안에서 행동한다.
> 초기 구축 사양은 별도 `STRUCTURE.md` 가 담당한다. 본 문서는 **저장소가 살아 있는 동안의 행동 규약**이다.
> 본 저장소는 plugin `{{PLUGIN_NAME}}` 의 한 인스턴스다. 인스턴스 변수는 `project.config.md` 가 단일 진실원.

---

## 1. 이 저장소가 무엇인가

{{PROJECT_NAME}}(언리얼 엔진 게임 프로젝트)의 **전용 지식 저장소**.
{{PROJECT_NAME}} 코드를 raw 로 삼아, 기능별로 정제된 마크다운 페이지를 쌓는다.

이 저장소는 다음 두 워크플로우의 접점이다.

- **워크플로우 1 — 인덱서**: {{PROJECT_NAME}} 코드 → 이 저장소에 **쓰기** (`/ingest` 스킬)
- **워크플로우 2 — 생성기**: 기획서 → 이 저장소 **읽기** → {{PROJECT_NAME}} 에 코드 파일 쓰기 (analyze-spec → locate → plan → implement)

본 저장소는 그 자체로 **코드를 생성하지 않는다**. 지식을 보관·제공할 뿐이다.

---

## 2. 신성한 규약 (절대 깨지 않는다)

### 2.1 mcwiki 는 읽기 전용
- mcwiki(`{{MCWIKI_SERVER}}` MCP)는 UE 일반 지식·함정 카탈로그다.
- 본 저장소 작업 중 mcwiki 도구는 **`read_index` / `read_page` / `list_pages` 만** 호출한다.
- `write_page`, `synthesis_seed` 등 mcwiki write 계열은 **절대 호출하지 않는다.**
- mcwiki 는 보편 지식, 본 저장소는 프로젝트 지식 — 경계를 흐리지 않는다.

### 2.2 추론보다 추출
- {{PROJECT_NAME}} 코드 주석에는 이미 설계 의도·`[[vault 링크]]`·함정이 들어 있을 수 있다.
- 페이지를 만들 때 **없는 걸 지어내지 말고, 주석에 있는 걸 정확히 옮긴다.**
- 추론이 필요하면 신뢰도를 낮추고(🟡/🔴) 그 사실을 본문에 명시한다.

### 2.3 신뢰도 태그는 거짓말하지 않는다
- 🟢 VAULT — 코드 주석·코드 구조에 **명확한 근거 있음**
- 🟡 PARTIAL — 코드에서 **합리적으로 추론** (주석 근거는 없음)
- 🔴 INFERRED — 코드 근거 없이 **일반 UE 지식으로 구성** (프로젝트 선례 없음)
- 한 항목이라도 🔴 가 핵심이면 페이지 전체 confidence 를 🟡 이하로 낮춘다.
- 🔴 와 ❓(검증 실패 링크)는 사용자가 가장 먼저 검토해야 할 부분이다.

### 2.4 index.md 는 항상 최신
- ingest 가 돌 때마다 마지막에 index.md 를 재생성한다.
- 카운트(entities/catalogs), Last ingested 날짜, 미인덱싱 큐를 빠짐없이 갱신.
- index.md 가 곧 생성기 locate 의 진입점이다. 여기가 낡으면 전 시스템이 낡는다.

### 2.5 빈칸 금지, 누락은 명시
- frontmatter 항목을 빈칸으로 두지 않는다. 모르면 `(unknown)` 같은 명시적 표기.
- 본문 섹션을 통째로 비우지 않는다. 정보가 없으면 "(주석에 근거 없음)" 같이 사실을 적는다.

### 2.6 {{PROJECT_NAME}} 쓰기는 모듈·폴더 단위로 끊어서
- 생성기가 {{PROJECT_NAME}} 에 코드 파일을 **쓸 때**, 한 번에 전체 변경을 쏟아내지 않는다.
- 영향 받는 **모듈** 또는 **폴더** 단위로 작업 후보를 나열하고, 사용자에게 **어느 단위를 먼저 처리할지 선택지를 제시**한다.
- 사용자가 선택한 **하나의 단위만 처리**하고, 결과(생성/수정된 파일 목록 + 신뢰도 태그)를 보고한 뒤 다음 단위의 선택을 받는다.
- 사용자가 "다음", "이건 건너뛰기", "여기까지" 같은 결정을 내릴 수 있도록 항상 다음 후보를 함께 제시한다.
- 이유: 통제권 유지, 폭주 방지, 단위별 검토 가능성. §3 의 plan 체크포인트와 함께 작동하는 추가 안전장치. 자동화(`-p`) 실행에서도 이 규약은 유효 — 단위 경계마다 사용자 입력을 대기한다.

---

## 3. 표준 워크플로우

### 3.1 ingest (코드 → 저장소)
`/ingest` 스킬의 SKILL.md 를 따른다. 요약 흐름:

1. raw·저장소 경로 확인
2. 인덱싱 대상 수집(증분)
3. 클래스별 대분류 판정 (§5)
4. entity 페이지 작성 (§4 양식)
5. vault_refs 링크 검증 (mcwiki read_page)
6. catalog 갱신
7. index.md 재생성
8. 체크포인트 — 🔴/❓ 항목을 사용자에게 보고하고 확인 후 다음 배치

### 3.2 생성기 locate (저장소 → 계획)
기획서의 각 요구사항에 대해 다음 순서로 fallback 한다.

1. **1차** — `index.md` 확인 → 관련 entity 페이지 읽기. 출처 🟢
2. **2차** — 페이지가 없으면 raw({{PROJECT_NAME}} 코드)를 직접 grep·읽기로 추론. 출처 🟡
3. **3차** — 프로젝트 선례도 없으면 mcwiki `read_page` 로 UE 표준 패턴 구성. 출처 🔴

locate-report 에는 각 항목의 출처 태그를 반드시 표기. plan 체크포인트에서 🔴 가 집중 검토 대상.

---

## 4. entity 페이지 양식 (운영 중 변경 시 절차)

양식 정식 정의는 STRUCTURE.md §4 에 있다. **양식을 바꿔야 할 일이 생기면**:

1. STRUCTURE.md §4 를 먼저 갱신
2. 기존 entity 들을 새 양식으로 마이그레이션 (점진적 허용)
3. ingest 스킬(SKILL.md, entity-template.md)을 새 양식에 맞게 동기화

양식은 두 워크플로우의 **계약**이므로 일방적으로 바꾸지 않는다.

---

## 5. 대분류 판정 (운영 중 빠른 참조)

> **단일 진실원** = [[categories]]. 본 §5 는 빠른 참조용 요약. 카테고리 신설·자동 분류 규칙·예약 카테고리의 권위 정의는 categories.md.

### 활성 카테고리 (6)

| 상속/특징 | 대분류 |
|---|---|
| `A...` 액터 + Actor 보조 UObject (Camera Mode 등) | **Actor** |
| `U...Component` 자손 + `UAnimInstance` 자손 | **Component** |
| `USubsystem` 자손 (Game/World/Tickable/Editor/Engine) | **Subsystem** |
| `I...Interface` (UINTERFACE 또는 plain C++) | **Interface** |
| `FTableRowBase` 자손 + UDataTable 로드·관리 역할 UObject | **DataTable** |
| `UAssetUserData` 자손 | **AssetUserData** |

### 예약 카테고리 (트리거 시 자동 신설)

| 카테고리 | 트리거 |
|---|---|
| **Asset** | U{{CLASS_PREFIX}}*Asset / UDataAsset / UPrimaryDataAsset (DataTable 자손 제외) 등장 시 |
| **GraphNode** | 그래프 노드 자손(U{{CLASS_PREFIX}}*_Node 등) 등장 시 |
| **BlueprintLibrary** | UBlueprintFunctionLibrary 자손 등장 시 |
| **Editor** | Editor 모듈의 F*/S*/UFactory/UEdGraphNode/UEdGraphSchema 등장 시 (정책 결정 필요) |

### 엣지 케이스 (카테고리 매칭 후에도 남는 항목)

- **다단 상속 USTRUCT**: 각각 독립 entity. 부모 함정은 "베이스 [[...]] 의 함정 적용"으로 링크.
- **미해결 의존**: 지어내서 페이지 만들지 말 것. 미인덱싱 큐에 등록.
- **분류 매칭 실패**: [[categories]] §신설 절차 1번 (사용자 보고) 발화. 강제 배치 / 신설 / 보류 3 선택.
- **역할 기준 판정 (베이스가 범용 UObject)**: UDataTable 컬렉션 관리 → DataTable, Actor 보조 → Actor. 본문에 "역할 기준 판정" 명시.

---

## 6. 자동화 실행 시 주의 (Cowork)

본 저장소를 다루는 작업이 **비대화형 자동화**(`claude -p`)로 실행될 때:

- 러너가 `--allowed-tools` 에 mcwiki 읽기 도구(read_index, read_page, list_pages)와 파일 도구를 **사전 명시**한다.
- `--disallowed-tools "ToolSearch"` 로 ToolSearch 지연 로딩을 차단한다.
- MCP 서버명은 underscore 표기 (dash 금지) — namespace 파싱 안정성.

자동화에서도 **체크포인트(반자동)** 는 유지한다. 생성기 plan 단계가 끝나면 plan.md 를 떨군 뒤 사용자 확인까지 대기. 완전 무인 실행은 검증된 기획서·반복 작업에만 허용.

---

## 7. 잘 작동하는 작업의 모습

- 추가/변경된 페이지는 frontmatter 가 빠짐없이 채워져 있다
- vault_refs 의 모든 링크가 mcwiki 에 실재한다 (또는 ❓ 로 표시되어 사용자에게 보고됐다)
- 🔴 항목과 ❓ 항목이 별도로 정리되어 사용자에게 전달됐다
- index.md 의 카운트·날짜·미인덱싱 큐가 최신이다
- 작업 결과 요약 보고에 "무엇이 바뀌었고, 무엇을 사용자가 확인해야 하는지" 가 명시되어 있다

---

## 8. 참조

- `STRUCTURE.md` — 초기 구축 사양, 양식 정식 정의
- `project.config.md` — 인스턴스 변수 단일 진실원
- `categories.md` — 대분류 권위 정의
- `BOOTSTRAP.md` — 이 저장소를 만든 부트스트랩 키트 (다음 인스턴스 생성용)
- `/ingest` 스킬 — 인덱싱 워크플로우의 상세 방법론
````

---

## 6. 템플릿 — `STRUCTURE.md` (초기 구축 사양)

````markdown
# STRUCTURE.md — {{VAULT_NAME}} 초기 구축 사양

> 이 문서는 **저장소를 처음 세울 때의 설계도**다.
> 운영 중 행동 규약은 `CLAUDE.md` 가 담당한다.
> 이 문서는 "무엇을, 어떤 모양으로 만들었는가"의 정식 정의(specification)다.

---

## 0. 용어

- **raw**: {{PROJECT_NAME}} 의 실제 소스 코드 (이 저장소 밖, 별도 UE 프로젝트)
- **저장소**: 이 {{VAULT_NAME}} 저장소 (정제된 지식 페이지들)
- **entity**: 클래스 하나에 대응하는 마크다운 페이지
- **catalog**: 한 대분류(예: Actor)에 속한 entity 들의 목록 페이지
- **index.md**: 저장소 전체의 진입점 (locate 의 시작점)

---

## 1. 저장소 목적

{{PROJECT_NAME}}(언리얼 게임 프로젝트)의 코드를 **기능별로 정제**하여,
기획서 기반 코드 생성(생성기)이 **읽고 근거로 삼을** 지식 베이스를 만든다.

raw 코드 → (인덱서) → 이 저장소 → (생성기) → {{PROJECT_NAME}} 새 코드

---

## 2. 최상위 레이아웃

```
{{VAULT_NAME}}/
├── CLAUDE.md          # 운영 헌법 (항상 가장 먼저 읽음)
├── STRUCTURE.md       # (이 문서) 초기 구축 사양
├── project.config.md  # 인스턴스 변수 단일 진실원
├── categories.md      # 대분류 권위 정의 (분류 규칙의 단일 진실원)
├── index.md           # 저장소 진입점 (생성기 locate 시작점)
├── catalogs/          # 대분류별 entity 목록
│   ├── actor.md
│   ├── component.md
│   ├── subsystem.md
│   ├── interface.md
│   ├── datatable.md
│   └── assetuserdata.md
├── entities/          # 클래스별 상세 페이지
│   └── (ingest 가 채움)
└── (그 외 메타 문서는 필요 시 추가)
```

---

## 3. raw 운영 방식

### 3.1 방식 A — 참조만 (기본 채택)
raw 코드는 이 저장소에 **복사하지 않는다.** {{PROJECT_NAME}} 의 실제 경로를 직접 읽는다.

- 장점: 중복 없음, 항상 최신
- 단점: {{PROJECT_NAME}} 경로(`{{RAW_ROOT}}`)가 유효해야 함
- 채택 이유: 저장소를 가볍게 유지, raw 변경이 자동 반영

### 3.2 경로 규약
raw 경로는 `project.config.md` 에 단일 진실원으로 둔다. 절대경로를 본문에 박지 않는다.

---

## 4. entity 페이지 양식 (정식 정의)

> 이 양식이 인덱서·생성기 두 워크플로우의 **계약**이다.

### 4.1 frontmatter (YAML)

```yaml
---
title: <클래스명>
category: <대분류>
confidence: <🟢|🟡|🔴>
source_path: <raw 기준 상대경로>
parent_class: <부모 클래스 or (none)>
vault_refs: [<mcwiki [[링크]] 목록>]
last_indexed: <YYYY-MM-DD>
---
```

### 4.2 본문 섹션 (순서 고정)

1. **역할 한 줄** — 이 클래스가 무엇을 하는가
2. **핵심 책임** — 주요 기능 bullet
3. **주요 멤버** — 필드·메서드 표 (이름, 타입, 설명)
4. **상호작용** — 다른 클래스와 어떻게 엮이는가 ([[entity 링크]])
5. **vault 함정** — mcwiki 에서 가져온 주의사항 ([[vault_refs]])
6. **출처 주석** — 코드 주석 인용 (있으면)

---

## 5. 대분류 판정

> 분류 규칙의 단일 진실원은 `categories.md`. 이 절은 요약. 생략 없이 categories.md 참조.

---

## 6. 초기 구축 순서

1. 디렉토리 스캐폴드 생성
2. project.config.md 작성 (인스턴스 변수)
3. categories.md 작성 (분류 규칙)
4. index.md 초기화
5. ingest 스킬로 entity 채우기 시작

---

## 7. 이 문서의 위치

STRUCTURE.md 는 구축 사양이다. 운영 규약(행동)은 CLAUDE.md, 인스턴스 값은 project.config.md, 분류 규칙은 categories.md 가 각각 단일 진실원이다. 양식 변경 시 §4 를 갱신하고 CLAUDE.md §4 절차를 따른다.
````

---

## 7. 템플릿 — `project.config.md` (인스턴스 변수)

````markdown
# project.config.md — {{VAULT_NAME}} 인스턴스 변수

> plugin `{{PLUGIN_NAME}}` 의 인스턴스 단일 진실원.
> 다른 프로젝트로 이 plugin 을 복제할 때 **이 파일만** 바꾸면 된다.

---

## 인스턴스 식별

| 키 | 값 |
|---|---|
| `project_name` | {{PROJECT_NAME}} |
| `vault_name` | {{VAULT_NAME}} |
| `plugin_name` | {{PLUGIN_NAME}} |
| `ue_version` | {{UE_VERSION}} |
| `mcwiki_server` | {{MCWIKI_SERVER}} |
| `class_prefix` | {{CLASS_PREFIX}} |

---

## raw 경로

| 키 | 값 |
|---|---|
| `raw_root` | {{RAW_ROOT}} |
| `raw_source_glob` | `{{RAW_SOURCE_GLOB}}` |

---

## 분류 카테고리 (요약)

상세는 `categories.md`. 활성 6 + 예약 4.

---

## mcwiki 연동

| 키 | 값 |
|---|---|
| `mcwiki_server` | {{MCWIKI_SERVER}} |
| `mcwiki_tools` | read_index, read_page, list_pages |

---

## 메모

- 이 파일은 인스턴스 고유값만 담는다. 행동 규약은 CLAUDE.md.
- raw_root 는 사용자 환경마다 다르므로 절대경로를 박지 않는다.
- 생성일: {{TODAY}}
````

---

## 8. 템플릿 — `categories.md` (대분류 권위 정의)

> 예약 카테고리 트리거는 프로젝트 클래스 접두사 `{{CLASS_PREFIX}}` 에 맞게 조정한다. 접두사가 없으면 KMC 예시를 참고해 프로젝트 등장 클래스에 맞게 다시 쓴다.

````markdown
# categories.md — {{VAULT_NAME}} 대분류 권위 정의

> 대분류(카테고리)에 관한 **단일 진실원**.
> CLAUDE.md §5 는 이 문서의 요약. 충돌 시 이 문서가 이긴다.

---

## 0. 이 문서의 권위 범위

- 어떤 카테고리가 존재하는가
- 클래스를 어떤 카테고리로 보낼 것인가 (판정 규칙)
- 새 카테고리를 언제, 어떻게 신설하는가

---

## 1. 활성 카테고리 (6)

| 카테고리 | 정의 | 대표 상속/특징 |
|---|---|---|
| **Actor** | 월드에 배치되는 액터 + 보조 UObject | `A...` 액터, Camera Mode 등 |
| **Component** | 액터에 부착되는 컴포넌트 | `U...Component`, `UAnimInstance` |
| **Subsystem** | 엔진/게임 라이프사이클 서비스 | `USubsystem` 자손 |
| **Interface** | 인터페이스 계약 | `I...Interface` |
| **DataTable** | 데이터 테이블 행 + 로더 | `FTableRowBase`, `UDataTable` 관리 |
| **AssetUserData** | 에셋에 부착되는 메타데이터 | `UAssetUserData` 자손 |

---

## 2. 예약 카테고리 (4, 트리거 시 신설)

| 카테고리 | 트리거 조건 |
|---|---|
| **Asset** | U{{CLASS_PREFIX}}*Asset / UDataAsset / UPrimaryDataAsset 등장 |
| **GraphNode** | U{{CLASS_PREFIX}}*_Node 등 그래프 노드 자손 등장 |
| **BlueprintLibrary** | UBlueprintFunctionLibrary 자손 등장 |
| **Editor** | Editor 모듈의 F*/S*/UFactory 등장 |

---

## 3. 카테고리 신설 절차

1. **트리거 감지** — 활성/예약 어디에도 매칭 안 되는 클래스 발견
2. **사용자 보고** — 후보 클래스와 제안 카테고리를 보고
3. **3선택 제시** — ① 기존 카테고리 강제 배치 ② 새 카테고리 신설 ③ 보류(미인덱싱 큐)
4. **확정 시 반영** — 본 문서 §1/§2 갱신 → catalogs/ 골격 추가 → CLAUDE.md §5 요약 동기화
````

---

## 9. 템플릿 — `index.md` (초기 상태, entity 0) 및 catalog 골격

> 부트스트랩 직후에는 entity 가 0 이다. ingest 가 돌면 §2.4 규약대로 재생성된다.

`index.md`:

````markdown
# index.md — {{VAULT_NAME}} 저장소 진입점

> 생성기 locate 의 시작점. ingest 가 돌 때마다 재생성된다.
> 이 파일이 낡으면 전 시스템이 낡는다 (CLAUDE.md §2.4).

---

## 저장소 통계

| 지표 | 값 |
|---|---|
| 총 entity | 0 |
| 활성 카테고리 | 6 |
| 예약(미사용) | 4 |
| Last ingested | (아직 없음) |

---

## 카테고리별 entity (locate 1차 진입)

### Actor (0)
(아직 없음 — ingest 대기)

### Component (0)
(아직 없음 — ingest 대기)

### Subsystem (0)
(아직 없음 — ingest 대기)

### Interface (0)
(아직 없음 — ingest 대기)

### DataTable (0)
(아직 없음 — ingest 대기)

### AssetUserData (0)
(아직 없음 — ingest 대기)

---

## 미인덱싱 큐

(비어 있음 — 첫 ingest 전)
````

각 `catalogs/<카테고리>.md` 골격 (6개, 카테고리명만 바꿔 반복):

````markdown
# catalog: Actor

> 이 대분류에 속한 entity 목록. ingest 가 갱신한다.

| entity | confidence | source_path |
|---|---|---|
| (아직 없음) | | |
````

---

## 10. 완료 기준 (Step 5 체크포인트에서 확인)

- [ ] `CLAUDE.md`, `STRUCTURE.md`, `project.config.md`, `categories.md`, `index.md` 5개가 생성됐다
- [ ] `catalogs/` 에 활성 6개 골격, `entities/.gitkeep` 존재
- [ ] 모든 파일에서 `{{...}}` 자리표시자가 **하나도 남지 않았다**
- [ ] `(unknown)` / 추측 변수 항목이 사용자에게 보고됐다
- [ ] "첫 ingest 진행 여부" 를 사용자에게 물었고 **여기서 멈췄다**

---

## 11. 자주 묻는 혼동

- **"왜 MCP 서버로 안 만드나?"** — 파일-디렉토리로 결정. ToolSearch 부담 회피 (원본 §8 계승).
- **"raw 를 복사하나?"** — 안 한다. 참조만(방식 A). `raw_root` 의 실제 경로를 읽는다.
- **"부트스트랩이 코드도 인덱싱하나?"** — 아니다. 골격 + 5개 문서까지만. ingest 는 별도.
- **"양식을 바로 바꿔도 되나?"** — 안 된다. CLAUDE.md §4 절차를 따른다.
- **"이 BOOTSTRAP.md 는 남겨두나?"** — 선택. 다음 인스턴스 생성을 위해 남겨두면 편하다.

<!-- END BOOTSTRAP KIT -->
