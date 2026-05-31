# KMCProject Knowledge — project.config

> 본 인스턴스의 변수 정의 단일 진실원. plugin `ue-project-knowledge` 의 `templates/project.config.md.template` 을 KMCProject 용으로 채운 결과.
> 다른 framework 문서들이 본 파일의 값을 참조한다 (`{{ VAR }}` placeholder). 운영 중 변경 시 framework 문서 동기 수정 책임.

## 필수 변수

| 변수 | 값 | 비고 |
|---|---|---|
| `PROJECT_NAME` | KMCProject | `KMCProject.uproject` 와 동일 |
| `KNOWLEDGE_REPO_NAME` | KMCKnowledge | 본 저장소 |
| `KNOWLEDGE_REPO_PATH` | `E:\KMCKnowledge` | 코드 저장소 *밖* |
| `CODE_REPO_PATH` | `E:\MCProject\KMCProject` | `.uproject` 폴더 |
| `ENTITY_NAME_PREFIX` | MC | `UMC*`, `AMC*`, `FMC*`, `IMC*`, `UMC*Component` 등 |
| `MODULE_API_PREFIXES` | `MCPLAYMODULE`, `MCEDITORMODULE`, `MCPLAYUMGMODULE` | 4개 모듈 (KMCProject 메인 모듈 외) |
| `UE_VERSION` | 5.7.4 | `KMCProject/Source/*.Target.cs` 의 IncludeOrderVersion = `Unreal5_7` |
| `MCWIKI_MCP_SERVER_NAME` | `MCWiki_-_UE_5_7_4_Knowledge_Vault` | underscore 사용 |

## 선택 변수

| 변수 | 값 | 비고 |
|---|---|---|
| `RAW_MODE` | A (참조만) | Cowork 에서 KMCProject 폴더를 *별도 마운트* — junction 불요 |
| `DEFAULT_BRANCH` | (미정) | git 사용 시 채움 |
| `OPERATOR_EMAIL` | sensr7086@naver.com | Cowork 사용자 |

---

## 본 인스턴스의 특이사항 (운영 중 발견된 patterns/edge cases)

### CP949 mojibake 주석
- 일부 컴포넌트(MCActorComponent, MCBouyancyComponent, MCMoveComponent, MCWaterPlaneComponent 등) 의 한국어 주석이 CP949 로 저장되어 UTF-8 으로 보면 깨짐.
- CLAUDE.md(코드 저장소 측, `E:\MCProject\KMCProject\CLAUDE.md`) 명시 — **임의 재인코딩 금지**.
- ingest 시 본문 함정 섹션에 "CP949 mojibake — 임의 재인코딩 금지" 명시.

### MCPlayModule 의 UnrealEd 의존
- `MCPlayModule.Build.cs` 가 `"UnrealEd"` 를 public dependency 로 가짐 → **Shipping 빌드 cook 불가**.
- 본 부채는 KMCProject CLAUDE.md 명시. ingest 의 MCGameSubsystem 등 entity 의 ⚠ 함정 섹션에 추가.

### 명칭 오타 누적
- 운영 중 발견된 오타: `Movemoent`/`Luanch`/`Infomation`/`Virual`/`Constranit`/`Cancle`/`Legde`/`Deteat`/`Overlab`/`DispalcementRto`/`MoveSpeedandle`/`MoveFlagandle` 등.
- BP 노출 변경 시 redirector 필요. 일괄 정리는 별도 작업으로.

### 분류 신설 이력 (categories.md §변경 이력 동기)
- 2026-05-29 (배치 3 직후) — Subsystem / Interface / AssetUserData 3 카테고리 정식 신설. 12 entity 재분류.
- 2026-05-29 (초기) — Actor / Component / DataTable 3 카테고리 도입.

### 본문 비활성 placeholder
- `UMCMeshProxyUserAssetData` — 헤더 본문 전체가 `/* ... */` 주석 안. 컴파일 비활성. 향후 본문 복구 / 파일 삭제 / 다른 위치 확인 결정 필요.

---

## plugin 측 참조

- plugin 위치: `E:\KMCKnowledge\plugin\ue-project-knowledge\` (개발 중)
- 설치 옵션: junction 또는 복사 → `%USERPROFILE%\.claude\plugins\ue-project-knowledge`
- plugin 의 templates 가 본 인스턴스의 framework 문서들의 출처.
