---
name: scaffold-ue-knowledge
description: >-
  ue-project-knowledge 플러그인의 templates/ 골격을 특정 UE 프로젝트에 맞춰 인스턴스화하는 스킬.
  사용자가 "XXXX 프로젝트에 맞춰 골격을 채워줘", "새 지식 저장소 만들어줘", "ue-project-knowledge 인스턴스화",
  "지식 저장소 골격 생성", "scaffold knowledge repo" 같은 요청을 할 때 사용한다. config 변수를 코드 저장소에서
  자동탐지하고 부족분만 사용자에게 물어, project.config.md + 5개 framework 문서 + catalogs 6개의 빈 골격을 생성한다.
  첫 ingest(코드→지식)는 하지 않는다 — 그건 ingest-ue-project 스킬의 몫이다.
---

# scaffold-ue-knowledge — UE 지식 저장소 골격 생성기

ue-project-knowledge 플러그인의 `templates/` 를 새 UE 프로젝트에 맞춰 **빈 골격**으로 찍어낸다.
산출물은 채워질 준비가 된 저장소 골격일 뿐, **실제 지식(entity)은 채우지 않는다.**

## 절대 규약

1. **골격만 만든다.** entity 페이지를 만들거나 코드를 읽어 지식을 추출하지 않는다. 그건 `ingest-ue-project` 의 일이다. 골격 생성이 끝나면 "이제 ingest 를 실행하세요" 로 사용자에게 넘긴다.
2. **mcwiki write 금지.** mcwiki(UE 일반 지식 MCP)는 이 작업에서 건드리지 않는다. `MCWIKI_MCP_SERVER_NAME` 변수는 *문자열로만* 기록한다.
3. **지식 저장소는 코드 저장소 *밖*.** `KNOWLEDGE_REPO_PATH` 가 `CODE_REPO_PATH` 의 하위 경로면 경고하고 사용자에게 재확인한다.
4. **빈칸 금지.** 변수를 모르면 빈 값으로 두지 말고 자동탐지 → 안 되면 사용자에게 질문. 그래도 없으면 `(unknown)` 으로 명시 기록.
5. **기존 저장소 덮어쓰기 금지.** 대상 경로에 이미 `CLAUDE.md` 나 `project.config.md` 가 있으면 중단하고 사용자에게 보고한다.
6. **literal `{{ VAR }}` 보존.** `project.config.md.template` 의 설명문에 예시로 들어 있는 `{{ VAR }}` 는 실제 placeholder 가 아니다 — 치환하지 않는다.

## 입력

요청 메시지에서 다음을 파악한다. 누락분은 아래 절차로 채운다.

- (필수) 대상 프로젝트 — 이름 또는 코드 저장소 경로
- (선택) 지식 저장소를 만들 경로
- (선택) 위 외의 변수 값 (사용자가 미리 준 경우)

## 변수 (project.config.md 의 단일 진실원)

| 변수 | 확보 방법 |
|---|---|
| `PROJECT_NAME` | `CODE_REPO_PATH` 의 `*.uproject` 파일명(확장자 제거). 없으면 폴더명. 최종 사용자 확인. |
| `KNOWLEDGE_REPO_NAME` | 기본 `<PROJECT_NAME>Knowledge`. |
| `KNOWLEDGE_REPO_PATH` | 사용자가 줌. 안 줬으면 질문. 코드 저장소 밖이어야 함. |
| `CODE_REPO_PATH` | 사용자가 줌(`.uproject` 가 있는 폴더). 안 줬으면 질문. |
| `ENTITY_NAME_PREFIX` | 소스에서 자동탐지 (아래). 못 찾으면 질문, 없으면 빈 값 허용. |
| `MODULE_API_PREFIXES` | 소스에서 `*_API` 매크로 자동탐지 (아래). |
| `UE_VERSION` | `.uproject` 의 `EngineAssociation`, 또는 `Source/*.Target.cs` 참조. 못 찾으면 질문. |
| `MCWIKI_MCP_SERVER_NAME` | 기본 `MCWiki_-_UE_<X>_<Y>_<Z>_Knowledge_Vault` (UE_VERSION 기반, underscore). 사용자 확인. |
| `RAW_MODE` (선택) | 기본 `A` (참조만). |
| `DEFAULT_BRANCH` (선택) | `git -C <CODE_REPO_PATH> symbolic-ref --short HEAD` 시도. 안 되면 빈 값. |
| `OPERATOR_EMAIL` (선택) | 모르면 빈 값. |

### 자동탐지 방법 (CODE_REPO_PATH 에 대해)

bash 로 코드 저장소를 읽어 추출한다 (Windows 경로는 마운트 경로로 변환해 접근):

- **PROJECT_NAME / UE_VERSION**: `*.uproject` 를 찾아 파일명과 `EngineAssociation` 필드를 읽는다.
- **ENTITY_NAME_PREFIX**: 헤더에서 `class <MODULE>_API [UAF]<Prefix>...` 패턴을 grep 해 가장 빈번한 클래스 접두사(U/A/F/I 다음 1~3 대문자)를 추린다. 예: `UMC`, `AMC` → 접두사 `MC`.
- **MODULE_API_PREFIXES**: `#define .*_API` 또는 `class [A-Z0-9]+_API` 에서 `*_API` 매크로 이름들을 수집해 콤마로 연결. 예: `MCPLAYMODULE, MCEDITORMODULE`.

탐지 결과는 사용자에게 "이렇게 탐지했습니다" 로 제시하고, **확신이 낮은 값(접두사·버전)은 확인을 받는다.** 필수 값 중 탐지 실패분은 `AskUserQuestion` 으로 모아서 한 번에 질문한다.

## 절차

0. **플러그인 templates 위치 확인.** 이 스킬이 속한 플러그인의 `templates/` 디렉토리를 찾는다 (`CLAUDE.md.template`, `STRUCTURE.md.template`, `categories.md.template`, `index.md.template`, `project.config.md.template`, `catalogs/_skeleton.md.template`). 못 찾으면 사용자에게 플러그인 경로를 묻는다.

1. **변수 확보.** 입력 + 자동탐지 + 질문으로 위 표의 모든 필수 변수를 확정. 선택 변수는 알면 채우고 모르면 비운다.

2. **대상 경로 안전 점검.** `KNOWLEDGE_REPO_PATH` 가 (a) 코드 저장소 밖인지, (b) 기존 골격이 없는지 확인. 위반 시 중단·보고.

3. **디렉토리 생성.** `KNOWLEDGE_REPO_PATH` 아래 `catalogs/`, `entities/` 생성. `entities/.gitkeep` 추가.

4. **project.config.md 작성.** `project.config.md.template` 를 복사하되, 표의 "값" 열 placeholder 를 확정 값으로 채운다. (이 파일은 sed 치환 대상이 *아니라* 직접 채우는 단일 진실원. 설명문의 `{{ VAR }}` 예시는 보존.)

5. **5개 framework 문서 치환 생성.** `CLAUDE.md`, `STRUCTURE.md`, `categories.md`, `index.md` (그리고 위 4의 config) 의 placeholder 를 일괄 치환해 저장소 루트에 쓴다. 치환 대상 placeholder:
   `{{ PROJECT_NAME }}`, `{{ KNOWLEDGE_REPO_NAME }}`, `{{ KNOWLEDGE_REPO_PATH }}`, `{{ CODE_REPO_PATH }}`, `{{ ENTITY_NAME_PREFIX }}`, `{{ MODULE_API_PREFIXES }}`, `{{ UE_VERSION }}`, `{{ MCWIKI_MCP_SERVER_NAME }}`.
   Windows 경로의 백슬래시는 sed 구분자 충돌·이스케이프에 주의 (`s|...|...|g` 형태 사용). 치환 후 결과 파일에 `{{ ` 가 남아 있지 않은지 grep 으로 검증한다 (config 의 의도된 `{{ VAR }}` 예시는 예외).

6. **catalogs 6개 생성.** 활성 6 카테고리 각각에 대해 `catalogs/_skeleton.md.template` 를 복사하며 `{{ CATEGORY }}`, `{{ KNOWLEDGE_REPO_NAME }}` 치환:
   `Actor → catalogs/actor.md`, `Component → catalogs/component.md`, `Subsystem → catalogs/subsystem.md`, `Interface → catalogs/interface.md`, `DataTable → catalogs/datatable.md`, `AssetUserData → catalogs/assetuserdata.md`. (파일명은 lowercase.)

7. **검증.** 생성된 모든 파일에서 (a) 미치환 `{{ ... }}` 잔존 여부(config 예외), (b) `index.md` 가 `0 entities / 6 catalogs` 인지, (c) `catalogs/` 에 정확히 6개 파일이 있는지 확인.

8. **보고하고 멈춘다.** 아래 형식으로 보고. **ingest 는 하지 않는다.**

## 완료 보고 형식

```
## 골격 생성 완료 — <PROJECT_NAME>

생성 위치: <KNOWLEDGE_REPO_PATH>

확정된 변수:
- PROJECT_NAME / KNOWLEDGE_REPO_NAME / ... (전부 나열, 자동탐지한 것은 [탐지], 사용자 입력은 [입력]으로 표기)

생성된 파일:
- project.config.md, CLAUDE.md, STRUCTURE.md, categories.md, index.md
- catalogs/ (6): actor, component, subsystem, interface, datatable, assetuserdata
- entities/ (빈 디렉토리)

⚠️ 확인 필요:
- 자동탐지 신뢰도가 낮았던 값 / (unknown) 으로 남긴 값

다음 단계:
- 이 저장소에서 ingest-ue-project 스킬을 실행하면 코드를 읽어 entity 를 채웁니다.
```
