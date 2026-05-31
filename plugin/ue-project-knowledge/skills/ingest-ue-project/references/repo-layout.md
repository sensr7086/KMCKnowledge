# 지식 저장소 디렉토리 레이아웃

저장소는 프로젝트 코드 **밖**의 독립 위치에 둔다.
raw = 프로젝트 코드. 정제 결과만 저장소에 쌓는다.
인스턴스별 경로·이름은 저장소 루트의 `project.config.md` 가 단일 진실원.

```
<프로젝트지식저장소>/
├── CLAUDE.md                 # 운영 헌법
├── STRUCTURE.md              # 구축 사양
├── project.config.md         # 인스턴스 변수 정의 (단일 진실원)
├── categories.md             # 카테고리 레지스트리 + 자동 분류 규칙 (단일 진실원)
├── index.md                  # 전체 카탈로그 · 생성기 locate 진입점 · 매 ingest 후 재생성
├── catalogs/                 # 카테고리당 1 파일
│   ├── actor.md
│   ├── component.md
│   ├── subsystem.md
│   ├── interface.md
│   ├── datatable.md
│   └── assetuserdata.md
├── entities/                 # 클래스별 정제 페이지
│   ├── XxxCharacter.md       # (접두사 뺀 파일명)
│   ├── XxxTableManager.md
│   └── XxxDataBase.md
└── raw/                      # 프로젝트 코드 사본 또는 심볼릭 참조 (출처)
```

## 명명 규약

- entity 파일명: 타입 접두사(`U`/`A`/`F`/`I`) 제거. `UXxxTableManager` → `XxxTableManager.md`, `FXxxDataBase` → `XxxDataBase.md`, `AXxxCharacter` → `XxxCharacter.md`. (`Xxx` 자리는 인스턴스의 `ENTITY_NAME_PREFIX`.)
- frontmatter `title` 에는 접두사 포함 정식명 유지.
- catalog 파일명: 카테고리 소문자, separator 없음 (`actor.md`, `assetuserdata.md`, `blueprintlibrary.md`).
- wiki 링크: `[[entities/XxxTableManager]]`, `[[catalogs/datatable]]` 형식.

## index.md 가 항상 담아야 할 것

- 헤더: Last ingested 날짜, entities/catalogs 카운트, raw 출처 경로
- 신뢰도 범례 (🟢🟡🔴)
- 활성 카테고리별 entity 나열 (각 항목에 신뢰도 태그 + 한 줄 상속 요약)
- 예약 카테고리 enumerate (트리거 안 됐어도 사용자가 인지)
- **미인덱싱 큐**: raw 에 있으나 아직 정제 안 된 클래스 목록 (카테고리별 그룹)

## mcwiki 와의 관계 (혼동 주의)

- **본 저장소(프로젝트 지식)**: 파일로 읽고 쓴다. 이 스킬이 write.
- **mcwiki (UE 일반 지식)**: MCP 로 **읽기만**. 링크 검증·UE 일반지식 참조 용도. 절대 write 안 함.
- 즉 읽기 대상이 둘인데 접근 방식이 다르다 — 파일 vs MCP.

## plugin 형태로 설치된 경우

- 스킬 본체는 `~/.claude/plugins/ue-project-knowledge/` 또는 마켓플레이스 설치 위치에 있다.
- 본 저장소의 `.claude/skills/` 는 옵셔널 (overrides 또는 deferred 로딩 회피용).
- references/ 파일은 plugin 측 — 운영 중 갱신 시 plugin 자체 업데이트 또는 인스턴스 측 override.
