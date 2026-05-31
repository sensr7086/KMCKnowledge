---
name: ingest-ue-project
description: Unreal Engine 프로젝트의 C++ 코드(raw)를 읽어 프로젝트 전용 지식 저장소(knowledge base)의 entity/catalog/index 마크다운으로 정제·적재하는 인덱서. 사용자가 "ingest", "인덱싱", "코드 스킬 생성", "지식 저장소 갱신", "프로젝트 코드를 위키로", "새 클래스 반영" 같은 말을 하거나, 프로젝트 코드가 추가·변경되어 지식 저장소를 최신화해야 할 때 반드시 사용한다. 기획서로 코드를 생성하는 작업(generate)과는 별개의 워크플로우다 — 이 스킬은 코드→지식 방향만 담당한다. 인스턴스 측 변수는 저장소 루트의 `project.config.md` 가 단일 진실원.
---

# ingest-ue-project — UE 프로젝트 코드 인덱서

UE 프로젝트의 C++ 코드를 읽어, 기능별로 정제된 지식 페이지를 프로젝트 지식 저장소에 쌓는다.
이 저장소는 나중에 "기획서 → 코드 생성" 워크플로우의 locate 단계가 읽는 1차 지식원이 된다.

## 이 스킬이 하는 일과 안 하는 일

- **한다**: raw 코드 읽기 → 카테고리 판정 → entity 페이지 작성 → catalog 갱신 → index 재생성.
- **안 한다**: 코드 생성, 기획서 해석, 엔진(mcwiki) 페이지 수정. mcwiki 는 **읽기만** (링크 검증용).

## 핵심 원칙 — 추론보다 추출

프로젝트 코드 주석에는 이미 설계 의도·`[[vault 링크]]`·함정이 풍부하게 적혀 있다 (잘 관리된 프로젝트의 경우).
**없는 걸 지어내지 말고, 주석에 있는 걸 정확히 옮기는 것이 최우선이다.**
주석에 근거가 있으면 🟢, 코드 구조에서 추론했으면 🟡, 주석/코드에 근거 없이 일반 UE 지식으로 메웠으면 🔴 로 표시한다.

---

## 워크플로우

### 0. 저장소 위치 확인

- raw 출처: 프로젝트 코드 루트 (저장소 루트의 `project.config.md` 의 `CODE_REPO_PATH` 참조).
- 저장소: 지식 저장소 루트 (프로젝트 *밖*, `KNOWLEDGE_REPO_PATH`).
- 구조: `index.md`, `categories.md`, `catalogs/<카테고리>.md`, `entities/<클래스>.md`, `raw/`.
- 정확한 디렉토리 규약은 `references/repo-layout.md` 참조.

### 1. 대상 수집

- 인덱싱할 `.h`/`.cpp` 쌍을 수집한다. 보통 헤더가 정보의 중심, cpp 는 구현 보강.
- 이미 인덱싱된 것은 `index.md` 의 카탈로그와 비교 — 변경된 것만 재처리(증분).
- 처음이면 전체. 클래스가 많으면 카테고리·모듈·폴더 단위로 끊어 진행하고 사용자에게 진척을 보고.

### 2. 클래스별 카테고리 판정

**단일 진실원** = 저장소 루트의 `categories.md`. 본 단계의 판정 규칙·신설 절차·예약 카테고리 모두 그 문서를 우선 참조.

활성 카테고리 + 예약 카테고리 의 자동 분류 규칙(매핑 순서 포함)은 `categories.md` §자동 분류 규칙 참조. 빠른 요약:

| 판정 조건 | 대분류 |
|---|---|
| 헤더에 `UINTERFACE(...)` + `I...Interface` 페어 또는 plain `class I...` | **Interface** |
| `: public USubsystem` 자손 | **Subsystem** |
| `: public UAssetUserData` | **AssetUserData** |
| `A...` 액터 + Actor 보조 UObject (Camera Mode 등) | **Actor** |
| `U...Component` 자손 + `UAnimInstance` | **Component** |
| `FTableRowBase` 자손 + UDataTable 관리 역할 UObject | **DataTable** |
| (예약) UObject 자산 / UDataAsset / UPrimaryDataAsset | **Asset** |
| (예약) 프로젝트 고유 그래프 콘텐츠 노드 | **GraphNode** |
| (예약) UBlueprintFunctionLibrary 자손 | **BlueprintLibrary** |
| (예약) UEdGraphNode / UEdGraphSchema / UFactory / S*Widget | **Editor** |

- 예약 카테고리는 첫 트리거 시 사용자 확인 후 `categories.md` 의 §신설 절차 4단계 (registry 추가 → catalog 골격 → classification.md 동기 → 영향 entity 재분류 → index 갱신) 발화.
- 분류 매칭 실패 시 사용자 보고 → 강제 배치(분류 한계 본문 명시) / 신설 / 보류 3 선택.
- 역할 기준 판정(베이스가 범용 UObject)은 `references/classification.md` §역할 기준 판정 참조.
- 인터페이스(`I...Interface`), 에셋, 다단 상속 등 엣지 케이스는 `references/edge-cases.md` 참조.

### 3. entity 페이지 작성

각 클래스를 `entities/<클래스명>.md` 로 작성한다. 양식과 작성 규칙은 **`references/entity-template.md` 를 반드시 먼저 읽고** 그대로 따른다. 요점:

- **frontmatter** 가 두 워크플로우의 계약이다. `category`, `base_class`, `module`, `confidence`, `source_paths`, `vault_refs`, `last_ingested` 를 빠짐없이 채운다.
- **`vault_refs` 는 코드 주석의 `[[...]]` 링크를 추출**한다. 새로 추론한 링크를 넣을 때는 본문에서 🔴 로 구분.
- **⚠ 함정 섹션**은 주석의 함정 기록(dangling, phantom header, UHT 거부 등)을 옮긴다. 이게 생성기의 안전장치이므로 누락 금지.
- 주석의 "설계 결정" 블록은 "따르는 패턴" 섹션으로 옮긴다.

### 4. vault 링크 검증 (mcwiki 읽기)

- `vault_refs` 에 넣은 각 `[[concepts/...]]` `[[synthesis/...]]` 등이 mcwiki 에 실제 존재하는지 확인.
- 효율적인 검증: 카테고리별로 `list_pages(kind)` 1회 호출 후 슬러그 매칭. 개별 `read_page` 는 의심 항목에만.
- 존재하지 않으면(오타·구버전 슬러그) 본문에 ❓ 로 표기하고 사용자에게 보고. 임의로 비슷한 페이지로 바꾸지 말 것.
- mcwiki 접근 방법·주의는 `references/mcwiki-access.md` 참조.

### 5. catalog 갱신

- 해당 카테고리의 `catalogs/<카테고리>.md` 에 이 클래스를 등록(베이스/관리자/구현체 + 페어 카탈로그 링크).
- 카테고리 공통 함정이 있으면 catalog 의 공통 함정 섹션에 모은다.

### 6. index.md 재생성

- 전체 entity/catalog 를 카탈로그로 다시 나열. 신뢰도 태그(🟢🟡🔴) 표시.
- 헤더의 카운트(entities/catalogs 수)와 `Last ingested` 날짜 갱신.
- raw 에 있으나 아직 미정제인 클래스는 "미인덱싱 큐"에 남긴다.
- index 는 생성기 locate 의 진입점이므로 **항상 최신**이어야 한다.

### 7. 체크포인트 (반자동)

- ingest 결과(새로 만든/바꾼 페이지 목록 + 🔴/❓ 표시 항목)를 사용자에게 요약 보고한다.
- 특히 🔴(근거 없이 메운 것)와 ❓(검증 실패 링크)를 **집중 검토 대상**으로 제시.
- 사용자 확인 후 다음 배치로 진행.
- 이전 배치의 entity 들이 새 batch 의 entity 를 "(미인덱싱)" 으로 표기했으면, 이번 ingest 종료 직전에 `[[entities/...]]` 로 승격.

---

## 참조 파일

- `references/entity-template.md` — entity 페이지 양식 + 작성 규칙 (작성 전 필독)
- `references/repo-layout.md` — 디렉토리 구조·파일 명명 규약
- `references/classification.md` — 카테고리 자동 분류 규칙·예시·역할 기준 판정
- `references/edge-cases.md` — 다단 상속·미해결 의존·본문 비활성·분류 매칭 실패 처리
- `references/mcwiki-access.md` — mcwiki 읽기 방법·링크 검증 절차

## 신뢰도 태그 (전 페이지 공통)

- 🟢 VAULT — 코드 주석·코드 구조에 명확한 근거 있음 (정제·검증됨)
- 🟡 PARTIAL — 코드에서 합리적으로 추론 (주석 근거는 없음)
- 🔴 INFERRED — 코드 근거 없이 일반 UE 지식으로 구성 (선례 없음)
