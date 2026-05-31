---
name: ingest-kmc
description: KMCProject 의 C++ 코드(raw)를 읽어 KMCKnowledge 저장소의 entity/catalog/index 마크다운으로 정제·적재하는 인덱서. 사용자가 "ingest", "인덱싱", "코드 스킬 생성", "지식 저장소 갱신", "프로젝트 코드를 위키로", "새 클래스 반영" 같은 말을 하거나, KMCProject 코드가 추가·변경되어 KMCKnowledge 를 최신화해야 할 때 반드시 사용한다. 기획서로 코드를 생성하는 작업(generate)과는 별개의 워크플로우다 — 이 스킬은 코드→지식 방향만 담당한다.
---

# ingest-kmc — KMCProject 코드 인덱서

KMCProject 의 C++ 코드를 읽어, 기능별로 정제된 지식 페이지를 KMCKnowledge 저장소에 쌓는다.
이 저장소는 나중에 "기획서 → 코드 생성" 워크플로우의 locate 단계가 읽는 1차 지식원이 된다.

## 이 스킬이 하는 일과 안 하는 일

- **한다**: raw 코드 읽기 → 대분류 판정 → entity 페이지 작성 → catalog 갱신 → index 재생성.
- **안 한다**: 코드 생성, 기획서 해석, 엔진(mcwiki) 페이지 수정. mcwiki 는 **읽기만** (링크 검증용).

## 핵심 원칙 — 추론보다 추출

KMCProject 코드 주석에는 이미 설계 의도·`[[vault 링크]]`·함정이 풍부하게 적혀 있다.
**없는 걸 지어내지 말고, 주석에 있는 걸 정확히 옮기는 것이 최우선이다.**
주석에 근거가 있으면 🟢, 코드 구조에서 추론했으면 🟡, 주석/코드에 근거 없이 일반 UE 지식으로 메웠으면 🔴 로 표시한다.

---

## 워크플로우

### 0. 저장소 위치 확인
- raw 출처: KMCProject 코드 루트 (예: `E:\MCProject\KMCProject`)
- 저장소: KMCKnowledge 루트 (프로젝트 밖, 예: `E:\MCProject\KMCKnowledge`)
- 구조: `index.md`, `catalogs/<대분류>.md`, `entities/<클래스>.md`, `raw/`
- 정확한 디렉토리 규약은 `references/repo-layout.md` 참조.

### 1. 대상 수집
- 인덱싱할 `.h`/`.cpp` 쌍을 수집한다. 보통 헤더가 정보의 중심, cpp 는 구현 보강.
- 이미 인덱싱된 것은 `index.md` 의 카탈로그와 비교 — 변경된 것만 재처리(증분).
- 처음이면 전체. 클래스가 많으면 대분류별로 끊어 진행하고 사용자에게 진척을 보고.

### 2. 클래스별 대분류 판정 (기계적)
상속 베이스와 역할로 판정한다. 규칙:

| 판정 조건 | 대분류 |
|---|---|
| `A...` 또는 `: public ACharacter/AActor/APawn...` | **Actor** |
| `U...Component` 또는 `: public U...Component` | **Component** |
| `: public FTableRowBase` (또는 그 자손) / `UDataTable` 관리·로드 역할 | **DataTable** |

- 판정이 모호하면 (예: UObject 인데 역할이 애매) 사용자에게 묻거나, 가장 가까운 대분류 + 본문에 근거를 남긴다.
- 인터페이스(`I...Interface`), 에셋(`U...Asset`) 등 위 셋에 안 맞는 것은 `references/edge-cases.md` 의 처리 지침을 따른다.

### 3. entity 페이지 작성
각 클래스를 `entities/<클래스명>.md` 로 작성한다. 양식과 작성 규칙은 **`references/entity-template.md` 를 반드시 먼저 읽고** 그대로 따른다. 요점:

- **frontmatter** 가 두 워크플로우의 계약이다. `category`, `base_class`, `module`, `confidence`, `source_paths`, `vault_refs`, `last_ingested` 를 빠짐없이 채운다.
- **`vault_refs` 는 코드 주석의 `[[...]]` 링크를 추출**한다. 새로 추론한 링크를 넣을 때는 본문에서 🔴 로 구분.
- **⚠ 함정 섹션**은 주석의 함정 기록(dangling, phantom header, UHT 거부 등)을 옮긴다. 이게 생성기의 안전장치이므로 누락 금지.
- 주석의 "설계 결정" 블록은 "따르는 패턴" 섹션으로 옮긴다.

### 4. vault 링크 검증 (mcwiki 읽기)
- `vault_refs` 에 넣은 각 `[[concepts/...]]` `[[synthesis/...]]` 등이 mcwiki 에 실제 존재하는지 `read_page` 로 확인.
- 존재하지 않으면(오타·구버전 슬러그) 본문에 ❓ 로 표기하고 사용자에게 보고. 임의로 비슷한 페이지로 바꾸지 말 것.
- mcwiki 접근 방법·주의는 `references/mcwiki-access.md` 참조.

### 5. catalog 갱신
- 해당 대분류의 `catalogs/<대분류>.md` 에 이 클래스를 등록(베이스/관리자/구현체 구분).
- 대분류 공통 함정이 있으면 catalog 의 공통 함정 섹션에 모은다.

### 6. index.md 재생성
- 전체 entity/catalog 를 카탈로그로 다시 나열. 신뢰도 태그(🟢🟡🔴) 표시.
- 헤더의 카운트(entities/catalogs 수)와 `Last ingested` 날짜 갱신.
- raw 에 있으나 아직 미정제인 클래스는 "미인덱싱 큐"에 남긴다.
- index 는 생성기 locate 의 진입점이므로 **항상 최신**이어야 한다.

### 7. 체크포인트 (반자동)
- ingest 결과(새로 만든/바꾼 페이지 목록 + 🔴/❓ 표시 항목)를 사용자에게 요약 보고한다.
- 특히 🔴(근거 없이 메운 것)와 ❓(검증 실패 링크)를 **집중 검토 대상**으로 제시.
- 사용자 확인 후 다음 배치로 진행.

---

## 참조 파일
- `references/entity-template.md` — entity 페이지 양식 + 작성 규칙 (작성 전 필독)
- `references/repo-layout.md` — 디렉토리 구조·파일 명명 규약
- `references/classification.md` — 대분류 판정 상세 규칙·예시
- `references/edge-cases.md` — 인터페이스·에셋·다단상속·미해결 의존 처리
- `references/mcwiki-access.md` — mcwiki 읽기 방법·링크 검증 절차

## 신뢰도 태그 (전 페이지 공통)
- 🟢 VAULT — 코드 주석·코드 구조에 명확한 근거 있음 (정제·검증됨)
- 🟡 PARTIAL — 코드에서 합리적으로 추론 (주석 근거는 없음)
- 🔴 INFERRED — 코드 근거 없이 일반 UE 지식으로 구성 (선례 없음)
