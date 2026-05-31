# mcwiki 접근 및 링크 검증

mcwiki(UE 일반 지식 Knowledge Vault, MCP 서버명은 인스턴스 `project.config.md` 의 `MCWIKI_MCP_SERVER_NAME` 참조 — 예: `MCWiki_-_UE_5_7_4_Knowledge_Vault`) 는 UE 일반 지식·함정 카탈로그를 담은 **MCP 서버**다.
이 스킬에서는 **읽기 전용** — vault_refs 링크가 실재하는지 검증하는 용도.

## 사용 도구 (읽기만)

- `read_index` — 전체 카탈로그. 어떤 슬러그가 있는지 확인.
- `read_page(kind, slug)` — 특정 페이지 본문. kind ∈ sources/entities/concepts/synthesis/meta.
- `list_pages(kind)` — 한 종류의 슬러그 목록.
- **write 계열(write_page 등) 은 이 스킬에서 절대 호출하지 않는다.** mcwiki 는 UE 일반지식 저장소이지 프로젝트 지식 저장소가 아니다.

## 링크 검증 절차

### 효율적인 일괄 검증 (권장)

1. entity 작성 중 코드 주석에서 `[[concepts/X]]`, `[[synthesis/Y]]`, `[[sources/Z]]`, `[[entities/W]]`, `[[meta/V]]` 형태를 추출. 배치 단위로 모음.
2. 4 kind(concepts/entities/sources/synthesis) 각각 `list_pages(kind)` 1회 호출 → 슬러그 목록 캐싱.
3. 추출한 슬러그가 캐싱된 목록에 있는지 매칭.
4. 매칭 → vault_refs 에 유지.
5. 매칭 실패(없는 슬러그) → 본문에 ❓ 표기 + 사용자 보고. **비슷한 다른 페이지로 임의 치환 금지.**

### 개별 검증 (소수 슬러그 또는 의심 항목)

1. 슬러그의 kind 와 이름을 분리 (`concepts/UE-FStructProperty-Cast-Type-Safety` → kind=concepts, slug=UE-FStructProperty-Cast-Type-Safety).
2. `read_page(concepts, "UE-FStructProperty-Cast-Type-Safety")` 로 존재 확인.

## 자동화 시 주의 (ToolSearch)

- Cowork 자동 실행에서는 mcwiki MCP 도구가 deferred 로딩될 수 있다.
- 러너가 `--allowed-tools` 에 mcwiki 읽기 도구(read_index/read_page/list_pages)를 사전 명시하고 `ToolSearch` 를 차단하는 것을 전제로 한다.
- MCP 서버명은 underscore 표기 (dash 금지) — namespace 파싱 안정성. 인스턴스 `MCWIKI_MCP_SERVER_NAME` 변수가 underscore 인지 확인.
- 상세 배경은 mcwiki 의 `[[concepts/Claude-Code-Cowork-ToolSearch-Bypass]]` 참조.

## UE 버전 호환성

- mcwiki MCP 서버는 UE 특정 버전의 vault. 인스턴스의 `UE_VERSION` 변수와 일치해야 함.
- 프로젝트 UE 버전이 mcwiki vault 와 다르면 — 예: 프로젝트 5.5.4 vs vault 5.7.4 — vault_refs 에서 API/슬러그 차이 가능성.
- 운영 중 vault 슬러그가 갱신되거나 페이지가 사라질 수 있다. 검증 없이 옛 슬러그를 박아넣지 말 것 — 반드시 list_pages / read_page 로 실재 확인.

## 주의

- mcwiki 슬러그는 자주 갱신된다. 검증 없이 옛 슬러그를 박아넣지 말 것.
- 대량 vault_refs 가 의심스러우면 (예: 옛 인덱스에서 복사한 페이지) — 일괄 list_pages 검증으로 사라진 슬러그 식별.
- 검증 실패 슬러그는 ❓ 표기 + 본문에 "구버전 슬러그 가능 — vault 갱신 확인 필요" 메모.
