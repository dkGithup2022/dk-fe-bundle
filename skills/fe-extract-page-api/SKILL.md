---
name: fe-extract-page-api
description: 화면 export 를 페이지 단위로 읽어 각 페이지가 부르는 API 를 뽑고, src/types/api/{domain}.ts 타입, handlers.ts mock 항목, src/lib/{domain}-api.ts 호출 함수를 만든다.
argument-hint: "[디자인 export 폴더]"
---

# 페이지별 API 추출

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

페이지마다 어떤 읽기와 쓰기가 필요한지 뽑아 엔드포인트로 정리하고, 페이지 스킬이 바로 부를 수 있는 호출 함수를 만드는 것이 목적입니다. BE 스펙이 있으면 경로와 필드명은 BE 를 따르고, 없으면 FE 제안으로 만들어 BE 확정 후 맞춥니다.

### 입력 & 출력

입력:
- 사용자가 지정한 디자인 export 폴더. 화면 `*.dc.html` 과 와이어프레임의 설명 (`dv-note`, 동작과 로그인 조건이 적혀 있다). `support.js` 는 읽지 않는다.
- BE API 스펙 (있으면).
- `src/types/data/` (fe-generate-data-types 산출물). 없으면 데이터 스킬(fe-extract-data-inventory → fe-extract-data-spec → fe-generate-data-types)을 먼저 실행한다.
- `src/lib/*-data.ts` 의 mock 레코드와 조회 함수. 없으면 이 스킬이 만든다.
- `src/types/api/`, `src/lib/api/mock/handlers.ts`, `src/lib/*-api.ts` 의 현재 내용.
- 대화에서 사용자가 정한 사항.

출력:
- `src/types/api/{domain}.ts`: 요청 파라미터와 응답 타입. 목록은 `PaginatedResponse<T>`.
- `src/lib/api/mock/handlers.ts`: 엔드포인트마다 항목 1개. 데이터는 `lib/{domain}-data.ts` 의 조회 함수를 부른다. 필요한 조회 함수가 없으면 그 파일에 추가한다.
- `src/lib/{domain}-api.ts`: 호출 함수. 함수마다 쓰는 페이지를 주석으로 단다.
- `src/lib/{domain}-api.test.ts`: mock 모드로 호출 함수를 한 번씩 부른다.

```ts
// src/types/api/group.ts
import type { PaginatedResponse, PaginationParams } from "./common";
import type { Group } from "@/types/data/domain/group";

/** BE 확정 전 초안. BE 스펙이 오면 이 파일을 맞추고 lib/group-api.ts 의 변환을 조정한다. */
export interface GroupListParams extends PaginationParams {
  category?: string;
  region?: string;
}
export type GroupListResponse = PaginatedResponse<Group>;
export type GroupResponse = Group;
export interface FollowGroupResponse {
  following: boolean;
  memberCount: number;
}
```

```ts
// src/lib/api/mock/handlers.ts 에 추가
{ method: "GET", path: "/api/groups", resolve: ({ params }) => getGroupPage(params) },
{ method: "GET", path: "/api/groups/:id", resolve: ({ pathParams }) => getGroup(Number(pathParams.id)) },
{ method: "POST", path: "/api/groups/:id/follow", resolve: ({ pathParams }) => toggleFollow(Number(pathParams.id)) },
```

```ts
// src/lib/group-api.ts
import { apiGet, apiPost } from "@/lib/api/client";
import type { QueryParams } from "@/lib/api/types";
import type { FollowGroupResponse, GroupListParams, GroupListResponse, GroupResponse } from "@/types/api/group";

/** QueryParams 는 undefined 를 허용하지 않으므로 선택 파라미터를 걷어낸다 */
function toQuery(params: object): QueryParams {
  return Object.fromEntries(Object.entries(params).filter(([, v]) => v !== undefined));
}

/** 페이지: 모임 찾기 */
export const fetchGroups = (params: GroupListParams) =>
  apiGet<GroupListResponse>("/api/groups", toQuery(params), { skipAuth: true });

/** 페이지: 모임 페이지, 이야기 뷰어 (하단 모임 카드) */
export const fetchGroup = (id: number) =>
  apiGet<GroupResponse>(`/api/groups/${id}`, undefined, { skipAuth: true });

/** 페이지: 모임 페이지, 모임 찾기 (GroupRow 팔로우 버튼). 로그인 필요 */
export const followGroup = (id: number) => apiPost<FollowGroupResponse>(`/api/groups/${id}/follow`);
```

```ts
// src/lib/group-api.test.ts
import { fetchGroups } from "./group-api";

beforeEach(() => vi.stubEnv("NEXT_PUBLIC_USE_MOCK", "true"));

it("모임 목록을 mock 으로 받는다", async () => {
  const res = await fetchGroups({ page: 0, size: 10 });
  expect(res.content.length).toBeGreaterThan(0);
});
```

### 권장 동작 순서

1. 기존 확인
   `src/types/api/` 의 export, `handlers.ts` 의 경로 목록, `src/lib/*-api.ts` 의 함수 목록을 뽑는다. 있는 것은 재사용한다.

2. 페이지 목록 만들기
   화면 라벨을 라우트 단위로 묶는다. 예: "모임 만들기 1 · 2 · 3 · 완료" 는 한 페이지, 시트와 모달은 그것을 여는 페이지에 속한다. 안이 여러 개면 확정안을 사용자에게 확인한다.

3. 페이지마다 표 작성
   - 읽기: 첫 진입에 필요한 데이터, 탭 · 더 보기 · 필터로 추가 로드하는 데이터
   - 쓰기: 버튼 · 폼 제출 · 토글이 바꾸는 데이터와 그 결과로 화면이 어떻게 바뀌는지
   - 조건: 로그인이 필요한 동작. 표시만 하고 인증 구현은 하지 않는다.

4. 엔드포인트로 합치기
   리소스 기준으로 정리한다. 여러 페이지가 같은 데이터를 다른 조건으로 읽으면 하나의 목록 API 에 파라미터를 둔다. 첫 진입 데이터는 페이지당 호출 1~2개로 맞춘다. BE 스펙이 있으면 그 경로와 필드명을 쓴다.

5. 사용자 확인
   페이지 × 엔드포인트 표를 보여주고 확인받는다.

6. 파일 쓰기
   타입 → handlers 항목 → 조회 함수 (필요하면 `lib/{domain}-data.ts` 에 추가) → 호출 함수 → 테스트 순서. 쓰기 API 의 mock 은 메모리 배열을 바꿔 다음 읽기에 반영되게 한다.

7. 검증
   `npm run typecheck && npm run lint && npm run test:run`.

### 종료 조건

- [ ] 모든 페이지의 읽기와 쓰기가 어느 호출 함수에 대응된다
- [ ] 호출 함수마다 `types/api` 타입과 `handlers.ts` 항목이 있다
- [ ] 페이지가 `lib/{domain}-api.ts` 만 import 하면 되고, 경로 문자열이 그 밖에 없다
- [ ] 쓰기 mock 의 결과가 다음 읽기에 반영된다
- [ ] 검증 명령을 통과했다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- BE 스펙과 화면 동작이 다르다
- 페이지가 요구하는 데이터가 `types/data` 에 없다 (fe-extract-data-spec 으로 돌아갈지 정한다)
- 동작의 결과가 화면에 없다 (제출 후 어디로 가는지, 무엇이 바뀌는지 알 수 없다)
