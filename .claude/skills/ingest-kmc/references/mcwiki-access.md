# mcwiki 접근 및 링크 검증

mcwiki(UE 5.7.4 Knowledge Vault)는 UE 일반 지식·함정 카탈로그를 담은 **MCP 서버**다.
이 스킬에서는 **읽기 전용** — vault_refs 링크가 실재하는지 검증하는 용도.

## 사용 도구 (읽기만)
- `read_index` — 전체 카탈로그. 어떤 슬러그가 있는지 확인.
- `read_page(kind, slug)` — 특정 페이지 본문. kind ∈ sources/entities/concepts/synthesis/meta.
- `list_pages(kind)` — 한 종류의 슬러그 목록.
- **write 계열(write_page 등) 은 이 스킬에서 절대 호출하지 않는다.** mcwiki 는 UE 일반지식 저장소이지 프로젝트 지식 저장소가 아니다.

## 링크 검증 절차
1. entity 작성 중 코드 주석에서 `[[concepts/X]]` 형태를 추출.
2. 슬러그의 kind 와 이름을 분리 (`concepts/UE-FStructProperty-Cast-Type-Safety` → kind=concepts, slug=UE-FStructProperty-Cast-Type-Safety).
3. `read_page(concepts, "UE-FStructProperty-Cast-Type-Safety")` 로 존재 확인.
4. 성공 → vault_refs 에 유지.
5. 실패(없는 슬러그) → 본문에 ❓ 표기 + 사용자 보고. **비슷한 다른 페이지로 임의 치환 금지.**

## 자동화 시 주의 (ToolSearch)
- Cowork 자동 실행에서는 mcwiki MCP 도구가 deferred 로딩될 수 있다.
- 러너가 `--allowed-tools` 에 mcwiki 읽기 도구(read_index/read_page/list_pages)를 사전 명시하고 `ToolSearch` 를 차단하는 것을 전제로 한다.
- MCP 서버명은 underscore 표기 (dash 금지) — namespace 파싱 안정성.
- 상세 배경은 mcwiki 의 [[concepts/Claude-Code-Cowork-ToolSearch-Bypass]] 참조.

## 주의
- mcwiki 슬러그는 자주 갱신된다. 검증 없이 옛 슬러그를 박아넣지 말 것 — 반드시 read_page 로 실재 확인.
