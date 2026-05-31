# CLAUDE.md — KMCKnowledge 운영 헌법

> 이 저장소(KMCKnowledge)에서 작업을 시작하는 모든 에이전트는 **이 문서를 가장 먼저 읽고** 그 규약 안에서 행동한다.
> 초기 구축 사양은 별도 `STRUCTURE.md` 가 담당한다. 본 문서는 **저장소가 살아 있는 동안의 행동 규약**이다.
> 본 저장소는 plugin `ue-project-knowledge` 의 첫 인스턴스다. 인스턴스 변수는 `project.config.md` 가 단일 진실원.

---

## 1. 이 저장소가 무엇인가

KMCProject(언리얼 엔진 게임 프로젝트)의 **전용 지식 저장소**.
KMCProject 코드를 raw 로 삼아, 기능별로 정제된 마크다운 페이지를 쌓는다.

이 저장소는 다음 두 워크플로우의 접점이다.

- **워크플로우 1 — 인덱서**: KMCProject 코드 → 이 저장소에 **쓰기** (`/ingest` 스킬)
- **워크플로우 2 — 생성기**: 기획서 → 이 저장소 **읽기** → KMCProject 에 코드 파일 쓰기 (analyze-spec → locate → plan → implement)

본 저장소는 그 자체로 **코드를 생성하지 않는다**. 지식을 보관·제공할 뿐이다.

---

## 2. 신성한 규약 (절대 깨지 않는다)

### 2.1 mcwiki 는 읽기 전용
- mcwiki(UE 5.7.4 Knowledge Vault MCP)는 UE 일반 지식·함정 카탈로그다.
- 본 저장소 작업 중 mcwiki 도구는 **`read_index` / `read_page` / `list_pages` 만** 호출한다.
- `write_page`, `synthesis_seed` 등 mcwiki write 계열은 **절대 호출하지 않는다.**
- mcwiki 는 보편 지식, 본 저장소는 프로젝트 지식 — 경계를 흐리지 않는다.

### 2.2 추론보다 추출
- KMCProject 코드 주석에는 이미 설계 의도·`[[vault 링크]]`·함정이 풍부하게 들어 있다.
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

### 2.6 KMCProject 쓰기는 모듈·폴더 단위로 끊어서
- 생성기가 KMCProject 에 코드 파일을 **쓸 때**, 한 번에 전체 변경을 쏟아내지 않는다.
- 영향 받는 **모듈**(예: `MCPlayModule`) 또는 **폴더**(예: `Actor/`, `Component/`, `MCGame/`) 단위로 작업 후보를 나열하고, 사용자에게 **어느 단위를 먼저 처리할지 선택지를 제시**한다.
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
2. **2차** — 페이지가 없으면 raw(KMCProject 코드)를 직접 grep·읽기로 추론. 출처 🟡
3. **3차** — 프로젝트 선례도 없으면 mcwiki `read_page` 로 UE 표준 패턴 구성. 출처 🔴

locate-report 에는 각 항목의 출처 태그를 반드시 표기. plan 체크포인트에서 🔴 가 집중 검토 대상.

---

## 4. entity 페이지 양식 (운영 중 변경 시 절차)

양식 정식 정의는 STRUCTURE.md §4 에 있다. **양식을 바꿔야 할 일이 생기면**:

1. STRUCTURE.md §4 를 먼저 갱신
2. 기존 entity 들을 새 양식으로 마이그레이션 (한 번에 다는 곤란하면 점진적으로)
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
| **Asset** | UMCStoryAsset / UMCPartsAsset / UDataAsset / UPrimaryDataAsset (DataTable 자손 제외) 등장 시 |
| **GraphNode** | UMCStory_* / UMCParts_* / UMCCombo* 그래프 노드 자손 등장 시 |
| **BlueprintLibrary** | UBlueprintFunctionLibrary 자손 등장 시 |
| **Editor** | MCEditorModule 의 F*/S*/UFactory/UEdGraphNode/UEdGraphSchema 등장 시 (정책 결정 필요) |

### 엣지 케이스 (카테고리 매칭 후에도 남는 항목)

- **다단 상속 USTRUCT**: 각각 독립 entity. 부모 함정은 "베이스 [[...]] 의 함정 적용"으로 링크.
- **미해결 의존**: 지어내서 페이지 만들지 말 것. 미인덱싱 큐에 등록.
- **분류 매칭 실패**: [[categories]] §신설 절차 1번 (사용자 보고) 발화. 강제 배치 / 신설 / 보류 3 선택.
- **역할 기준 판정 (베이스가 범용 UObject)**: UDataTable 컬렉션 관리 → DataTable, Actor 보조 → Actor. 본문에 "역할 기준 판정" 명시.

> 이전 엣지 케이스였던 **인터페이스** 와 **에셋(UAssetUserData)** 은 2026-05-29 (배치 3 직후) 정식 카테고리화로 해소됨.

---

## 6. 자동화 실행 시 주의 (Cowork)

본 저장소를 다루는 작업이 **비대화형 자동화**(`claude -p`)로 실행될 때:

- 러너가 `--allowed-tools` 에 mcwiki 읽기 도구(read_index, read_page, list_pages)와 파일 도구를 **사전 명시**한다.
- `--disallowed-tools "ToolSearch"` 로 ToolSearch 지연 로딩을 차단한다.
- MCP 서버명은 underscore 표기 (dash 금지) — namespace 파싱 안정성.
- 배경 설명: mcwiki `[[concepts/Claude-Code-Cowork-ToolSearch-Bypass]]`

자동화에서도 **체크포인트(반자동)** 는 유지한다. 생성기 plan 단계가 끝나면 plan.md 를 떨군 뒤 사용자 확인까지 대기. 완전 무인 실행은 검증된 기획서·반복 작업에만 허용.

---

## 7. 잘 작동하는 작업의 모습

이 저장소에서 한 회의 작업이 잘 끝났다고 말할 수 있으려면:

- 추가/변경된 페이지는 frontmatter 가 빠짐없이 채워져 있다
- vault_refs 의 모든 링크가 mcwiki 에 실재한다 (또는 ❓ 로 표시되어 사용자에게 보고됐다)
- 🔴 항목과 ❓ 항목이 별도로 정리되어 사용자에게 전달됐다
- index.md 의 카운트·날짜·미인덱싱 큐가 최신이다
- 작업 결과 요약 보고에 "무엇이 바뀌었고, 무엇을 사용자가 확인해야 하는지" 가 명시되어 있다

---

## 8. 자주 묻는 혼동

- **"새 저장소를 mcwiki 처럼 MCP 서버로 만들면 안 되나?"** — 안 한다. 파일 디렉토리로 결정함. ToolSearch 부담 회피.
- **"코드 생성을 monolith 같은 도구로 하지 않나?"** — 안 한다. (나) 텍스트 코드 파일 생성. Cowork 파일 쓰기로 충분.
- **"raw 가 비어 있으면 ingest 못 도나?"** — 도는 방식이 다르다. raw 운영 방식 (A) 참조만 이면 KMCProject 실제 경로를 읽는다. STRUCTURE.md §3.4 참조.
- **"양식이 마음에 안 들면 바로 바꿔도 되나?"** — 안 된다. §4 절차를 따른다.

---

## 9. 참조

- `STRUCTURE.md` — 초기 구축 사양, 양식 정식 정의
- `/ingest` 스킬 (SKILL.md + references/) — 인덱싱 워크플로우의 상세 방법론
- mcwiki `[[meta/...]]` 페이지들 — vault 운영 철학의 원전 (참고용)
