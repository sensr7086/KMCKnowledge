# KMCKnowledge 디렉토리 레이아웃

저장소는 KMCProject **밖**의 독립 위치에 둔다 (예: `E:\MCProject\KMCKnowledge`).
raw = KMCProject 코드. 정제 결과만 저장소에 쌓는다.

```
KMCKnowledge/
├── index.md                 # 전체 카탈로그 · 생성기 locate 진입점 · 매 ingest 후 재생성
├── catalogs/
│   ├── actor.md             # 대분류별 목차 (베이스/관리자/구현체 구분)
│   ├── component.md
│   └── datatable.md
├── entities/
│   ├── MCCharacter.md       # 클래스별 페이지 (접두사 뺀 파일명)
│   ├── MCTableManager.md
│   └── MCDataBase.md
└── raw/                     # KMCProject 코드 사본 또는 심볼릭 참조 (출처)
```

## 명명 규약
- entity 파일명: 타입 접두사(`U`/`A`/`F`/`I`) 제거. `UMCTableManager` → `MCTableManager.md`, `FMCDataBase` → `MCDataBase.md`, `AMCCharacter` → `MCCharacter.md`
- frontmatter `title` 에는 접두사 포함 정식명 유지
- catalog 파일명: 대분류 소문자 (`actor.md`, `component.md`, `datatable.md`)
- wiki 링크: `[[entities/MCTableManager]]`, `[[catalogs/datatable]]` 형식

## index.md 가 항상 담아야 할 것
- 헤더: Last ingested 날짜, entities/catalogs 카운트, raw 출처 경로
- 신뢰도 범례 (🟢🟡🔴)
- 대분류별 entity 나열 (각 항목에 신뢰도 태그 + 한 줄 상속 요약)
- **미인덱싱 큐**: raw 에 있으나 아직 정제 안 된 클래스 목록

## mcwiki 와의 관계 (혼동 주의)
- **새 저장소(KMCKnowledge)**: 파일로 읽고 쓴다. 이 스킬이 write.
- **mcwiki**: MCP 로 **읽기만**. 링크 검증·UE 일반지식 참조 용도. 절대 write 안 함.
- 즉 읽기 대상이 둘인데 접근 방식이 다르다 — 파일 vs MCP.
