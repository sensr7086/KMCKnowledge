# ue-project-knowledge

UE 프로젝트의 **전용 지식 저장소(knowledge base)** 를 표준 골격으로 운영하기 위한 Claude Code 플러그인.

## 무엇을 해결하나

UE 프로젝트가 커지면 "이 프로젝트는 *실제로* 이렇게 짜여 있다" 라는 지식이 흩어진다. mcwiki(UE 일반 지식) 는 엔진 표준만 다루고, 프로젝트 고유의 클래스·패턴·함정은 코드 주석에 묻혀 있다. 이 플러그인은:

1. **인덱서 워크플로우** — 프로젝트 코드를 읽어 카테고리별로 정제된 entity 페이지를 생성·관리한다 (`/ingest-ue-project` 스킬).
2. **확장 가능한 카테고리 시스템** — 활성 6 (Actor / Component / Subsystem / Interface / DataTable / AssetUserData) + 예약 4 (Asset / GraphNode / BlueprintLibrary / Editor). 새 카테고리는 신설 절차 4단계로 자동 분류.
3. **mcwiki 와의 경계** — 본 저장소는 *프로젝트 지식*, mcwiki 는 *엔진 일반 지식*. vault_refs 링크로 연결. 자손 워크플로우(생성기 등) 가 두 출처를 fallback 으로 사용.

## 첫 인스턴스 — KMCProject

본 플러그인은 KMCProject 의 운영 경험에서 추출되었다. KMCKnowledge (`E:\KMCKnowledge`) 가 첫 인스턴스이며, 32 entity / 6 catalog / 0 ❓ / 0 🔴(실 클래스 기준) 의 검증된 상태를 유지한다. 본 plugin 의 templates 는 그 골격의 generic 화 산출물.

## 구조

```
ue-project-knowledge/
├── .claude-plugin/
│   └── plugin.json
├── README.md                          (이 문서)
├── skills/
│   └── ingest-ue-project/             # 인덱서 스킬 (코드 → 지식)
│       ├── SKILL.md
│       └── references/
│           ├── repo-layout.md          (저장소 명명·디렉토리 규약)
│           ├── classification.md       (카테고리 자동 분류 규칙)
│           ├── edge-cases.md           (엣지 케이스 처리)
│           ├── entity-template.md      (entity 페이지 양식)
│           └── mcwiki-access.md        (vault 링크 검증)
└── templates/                          # 인스턴스화 시 새 프로젝트 저장소로 복사
    ├── project.config.md.template      (변수 정의 — placeholder 의 단일 진실원)
    ├── CLAUDE.md.template              (운영 헌법)
    ├── STRUCTURE.md.template           (구축 사양)
    ├── categories.md.template          (카테고리 레지스트리)
    ├── index.md.template               (전체 카탈로그)
    └── catalogs/
        └── _skeleton.md.template       (catalog 골격)
```

## 설치

### 옵션 1: Junction (개발 중 권장)

플러그인 위치(예: 본 저장소 자체) → `%USERPROFILE%\.claude\plugins\` 로 junction:

```cmd
mklink /J %USERPROFILE%\.claude\plugins\ue-project-knowledge E:\KMCKnowledge\plugin\ue-project-knowledge
```

PowerShell:
```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\plugins\ue-project-knowledge" -Target "E:\KMCKnowledge\plugin\ue-project-knowledge"
```

### 옵션 2: 복사

```cmd
xcopy E:\KMCKnowledge\plugin\ue-project-knowledge %USERPROFILE%\.claude\plugins\ue-project-knowledge /E /I
```

### 옵션 3: 마켓플레이스 배포 (향후)

`.claude-plugin/plugin.json` 매니페스트를 갖춘 상태로 git 저장소화 후 마켓플레이스 등록.

## 새 프로젝트에서 사용

1. **저장소 위치 결정** — 예: `E:\MyProject\MyProjectKnowledge` (코드 저장소 *밖*).
2. **`project.config.md` 작성** — `templates/project.config.md.template` 을 복사 후 변수 채움:
   - `PROJECT_NAME = MyProject`
   - `KNOWLEDGE_REPO_PATH = E:\MyProject\MyProjectKnowledge`
   - `CODE_REPO_PATH = E:\MyProject\MyProject` (예)
   - `ENTITY_NAME_PREFIX = MP` (예: UMP*, AMP*, FMP*, IMP*) — 없으면 빈 값
   - `MODULE_API_PREFIXES = MYPROJECTMODULE`
   - `UE_VERSION = 5.7.4`
   - `MCWIKI_MCP_SERVER_NAME = MCWiki_-_UE_5_7_4_Knowledge_Vault`
3. **templates 의 나머지 5 파일을 새 저장소 루트에 복사** — `{{ ... }}` placeholder 가 들어 있음. sed 또는 find-replace 로 일괄 치환:
   ```bash
   for f in CLAUDE.md STRUCTURE.md categories.md index.md; do
     sed -e "s/{{ PROJECT_NAME }}/MyProject/g" \
         -e "s/{{ KNOWLEDGE_REPO_NAME }}/MyProjectKnowledge/g" \
         -e "s|{{ CODE_REPO_PATH }}|E:\\\\MyProject\\\\MyProject|g" \
         -e "s/{{ ENTITY_NAME_PREFIX }}/MP/g" \
         < templates/${f}.template > /path/to/new/repo/${f}
   done
   ```
4. **catalogs/ 6 파일 신설** — 활성 6 카테고리 각각의 골격을 `templates/catalogs/_skeleton.md.template` 으로 복사 후 `{{ CATEGORY }}` 치환.
5. **`/ingest-ue-project` 스킬 실행** — 첫 ingest 가 entity 들을 생성하고 카탈로그·index 를 갱신.

## 사용 예 (트리거)

```
ingest 진행해줘
새 클래스 반영해줘
프로젝트 코드를 위키로 옮겨줘
지식 저장소 갱신해줘
```

## 카테고리 신설 (운영 중)

새 entity 가 활성 카테고리에 안 맞으면 ingest 스킬이 사용자에게 보고 → 신설 승인 시 `categories.md` §신설 절차 4단계 자동 수행:

1. categories.md §활성 카테고리 표에 행 추가
2. catalogs/&lt;lowercase&gt;.md 골격 생성
3. references/classification.md 의 활성 카테고리 표 동기 갱신
4. 영향 entity 의 frontmatter `category` 재분류 + index.md 갱신

예약 카테고리 (Asset / GraphNode / BlueprintLibrary / Editor) 는 첫 트리거 시 자동 신설 후보로 제시.

## 워크플로우 2 (생성기) — 미구현

본 plugin 은 **인덱서 워크플로우(코드 → 지식)** 만 담당한다. 자손 워크플로우인 **생성기(기획서 → 코드)** 는 별도 plugin 으로 분리 예정:

- analyzer → locator → planner → implementer (+reviewer) 파이프라인
- 본 저장소의 index/entity 를 1차 fallback, mcwiki 를 2차 fallback, 추론을 3차 fallback 으로 사용

## 라이선스

(미정 — 추후 작성)

## 변경 이력

- **0.1.0 (2026-05-29)** — 초기 plugin 화. KMCProject 의 운영 경험에서 추출. 6 활성 + 4 예약 카테고리 + 신설 절차.
