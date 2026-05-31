# 엣지 케이스 처리

활성 카테고리(Actor/Component/Subsystem/Interface/DataTable/AssetUserData)와 예약 카테고리 매칭 후에도 남는 항목들. **단일 진실원** = 저장소 루트의 `categories.md`.

## 이전 엣지 케이스 (해소됨)

- **인터페이스 (`I...Interface`)** — 2026-05-29 (배치 3 직후) **Interface** 카테고리 정식 신설로 해소. UINTERFACE / plain C++ 양분.
- **에셋 (`U...AssetUserData`)** — 동일 시점 **AssetUserData** 카테고리 신설로 해소. primary 자산 (`U...Asset`) 은 예약 카테고리 **Asset** 신설 트리거 시 자동 분리.

## 현재 엣지 케이스

### 다단 상속 USTRUCT (DataTable)
- 예: `FMCData_Weapon : FMCData_Item : FMCDataBase`
- 각각 독립 entity 로 만들되, 상속 체인을 "소속/상속" 에 명시하고 부모를 "연관 entity" 로 링크.
- 베이스(FMCDataBase)의 ⚠ 함정은 자손 페이지에서 "베이스 [[entities/MCDataBase]] 의 함정 적용" 으로 참조 (중복 복사 대신 링크).

### 다단 상속 UObject (GraphNode 예약 카테고리)
- 예: `UMCParts_SetSkinnedMesh : UMCParts_SetMesh : UMCParts_Node`
- 본 예약 카테고리 첫 트리거 (Story / Parts / Combo 배치) 시 신설. 같은 처리 — 각 자손 독립 entity + 부모 함정 링크.

### 미해결 의존 (raw 에 아직 없는 클래스 참조)
- 예: MCCharacter 가 UMCPartsAsset 을 쓰는데 아직 인덱싱 전.
- "연관 entity" 에 `(미인덱싱 — <카테고리> 배치)` 표기 + `index.md` 의 미인덱싱 큐에 추가.
- 지어내서 페이지를 만들지 말 것. 큐에만 남기고 다음 배치에서 처리.
- 의존 entity 가 인덱싱되면 다음 ingest 에서 `(미인덱싱)` → `[[entities/...]]` 로 승격.

### 본문 비활성 (placeholder)
- 예: [[entities/MCMeshProxyUserAssetData]] — 헤더 본문이 `/* ... */` 주석 처리. 실제 컴파일되는 클래스 없음.
- 처리:
  - frontmatter `confidence: 🔴 INFERRED`.
  - 본문에 "현재 컴파일 안 되는 placeholder" 명시.
  - 의도된 본문(주석에서 추출)을 코드 블록으로 보존.
  - 사용자에게 (a) 본문 복구 / (b) 파일 삭제 / (c) 다른 위치 본문 확인 3 선택 제시.

### 분류 매칭 실패 (자동 분류 후)
- `categories.md` §자동 분류 규칙 의 top-down 매칭 모두 실패 시.
- 사용자 보고 — "이 entity 는 기존 N 활성 카테고리에 정확히 안 맞습니다. 가장 가까운: X. 예약 카테고리: Y. 신설 제안: Z."
- 사용자 선택 (강제 배치 / 신설 / 보류) — `categories.md` §신설 절차.

### 강제 배치 시 본문 의무
- 활성 카테고리에 강제 배치하되 분류 한계가 있는 경우 (예: MCStorySubSystem 이 Subsystem 신설 전에는 DataTable 강제 배치):
  - 본문 "소속/상속" 섹션에 **분류 한계 명시** — 어느 카테고리의 어떤 정의에 어긋나는지.
  - `confidence: 🟡 PARTIAL` 으로 낮춤.
  - 차기 카테고리 신설 시점에 재분류 검토 명시.

## 신설·재분류 절차 (요약)

상세는 `categories.md` §신설 절차. 4 단계:
1. 본 categories.md §활성 카테고리 표에 행 추가.
2. `catalogs/<lowercase>.md` 골격 생성.
3. 본 classification.md 의 활성 카테고리 표 동기 갱신.
4. 영향 entity 의 frontmatter `category` 재분류 + 본 catalog 등록 + `index.md` 갱신.
