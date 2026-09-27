---
name: fe-generate-api
description: 사용자가 검수한 docs/pages/{라우트 이름}/api.md 를 읽어 "새로 필요" 한 API 를 코드로 만든다. types/api 타입, mock 데이터와 조회 함수, handlers.ts 항목, lib/{domain}-api.ts 호출 함수, 테스트. 문서에 없는 것은 만들지 않는다.
argument-hint: "[라우트 이름]"
---

# API 구현

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

검수가 끝난 API 스펙 문서를 그대로 코드로 옮기는 것이 목적입니다. 판단은 하지 않습니다. 문서에 없는 엔드포인트나 필드를 더하지 않습니다. BE 가 없어도 페이지가 돌아가도록 mock 까지 만듭니다.

### 입력 & 출력

입력:
- `docs/pages/{라우트 이름}/api.md`. 상태가 "검수 완료" 여야 하고 "결정 필요" 가 비어 있어야 한다.
- `docs/pages/{라우트 이름}/data.md` 의 "API 데이터" 절. 타입 이름과 필드는 여기 것을 쓴다.
- `src/types/data/` 의 도메인 타입. mock 레코드는 이 타입으로 만든다.
- 이미 있는 것: `src/types/api/`, `src/lib/*-data.ts`, `src/lib/api/mock/handlers.ts`, `src/lib/*-api.ts`. 있는 것은 고치지 않는다.
- 화면 이미지 (mock 문구를 화면에서 가져올 때).

출력 (엔드포인트가 속한 도메인마다):
- `src/types/api/{domain}.ts`: 요청 파라미터 · 응답 타입. 머리에 "BE 확정 전 초안".
- `src/lib/{domain}-data.ts`: mock 레코드와 조회 함수. 레코드는 `types/data` 의 도메인 타입, 문구는 화면 것.
- `src/lib/api/mock/handlers.ts`: 엔드포인트마다 항목 하나. 조회 함수를 부른다.
- `src/lib/{domain}-api.ts`: 호출 함수. 함수마다 쓰는 페이지를 주석으로.
- `src/lib/{domain}-api.test.ts`: mock 모드로 호출 함수를 한 번씩 부른다. 쓰기는 다음 읽기에 반영되는지 본다.

```ts
// src/types/api/{domain}.ts
import type { PaginatedResponse, PaginationParams } from "./common";
import type { {Name} } from "@/types/data/domain/{name}";

/** BE 확정 전 초안. docs/pages/{라우트 이름}/api.md */
export interface {Name}ListParams extends PaginationParams {
  {field}?: {타입}; // {화면 근거}
}
export type {Name}ListResponse = PaginatedResponse<{Name}ListItem>;
```

```ts
// src/lib/api/mock/handlers.ts 에 추가
{ method: "GET", path: "/api/{resources}", resolve: ({ params }) => get{Name}Page({ ... }) },
{ method: "POST", path: "/api/{resources}/:id/{action}", resolve: ({ pathParams }) => {action}(Number(pathParams.id)) },
```

```ts
// src/lib/{domain}-api.ts
import { apiGet, apiPost, toQueryParams } from "@/lib/api/client";

/** 페이지: {라우트 이름} ({동작}) */
export const fetch{Name}s = (params: {Name}ListParams) =>
  apiGet<{Name}ListResponse>("/api/{resources}", toQueryParams(params), { skipAuth: true });
```

```ts
// src/lib/{domain}-api.test.ts
import { beforeEach, describe, expect, it, vi } from "vitest";
import { fetch{Name}s } from "./{domain}-api";

beforeEach(() => vi.stubEnv("NEXT_PUBLIC_USE_MOCK", "true"));

describe("{라우트 이름} API (mock)", () => {
  it("{동작}", async () => {
    const res = await fetch{Name}s({ page: 0, size: 10 });
    expect(res.content.length).toBeGreaterThan(0);
  });
});
```

### 작성 규칙

- api.md 의 "새로 필요" 만 만든다. "있음" 은 손대지 않는다. "API 아님" 은 건너뛴다.
- 인증 "불필요" 는 호출 함수에 `{ skipAuth: true }`. "필요" 는 붙이지 않는다.
- 선택 파라미터는 `toQueryParams` 로 걷어내서 넘긴다. QueryParams 는 undefined 를 허용하지 않는다.
- 응답 타입 이름은 api.md 와 data.md 에 적힌 것을 그대로 쓴다.

### mock 규칙

호출 함수마다 mock 응답을 같이 만든다. 템플릿은 `NEXT_PUBLIC_USE_MOCK=true` 일 때 `src/lib/api/mock/handlers.ts` 로 응답하므로, mock 이 없으면 페이지가 돌아가지 않는다.

- 위치: 레코드와 조회 함수는 `src/lib/{domain}-data.ts`. `handlers.ts` 에는 경로 항목만 두고 조회 함수를 부른다. 이미 있는 파일이면 거기에 보탠다.
- 레코드: `types/data` 의 도메인 타입으로 만든다. 엔티티마다 3개 안팎. 문구는 화면에 실린 것을 그대로 쓴다. 지어내지 않는다.
- 조회 함수: api.md 의 요청 파라미터를 그대로 받아 거르고 정렬한다. 목록은 `PaginatedResponse` 모양으로 돌려준다.
- 쓰기: 메모리 상태(`Set`, `Map`, 배열)를 바꿔서 다음 읽기에 반영되게 한다. 응답에는 api.md 대로 바뀐 값을 넣는다.
- 오류: api.md 의 동작에 "없으면", "거부한다" 가 있으면 `handlers.ts` 에서 `throw new ApiError(메시지, 상태 코드)`. 클라이언트가 그대로 받는다.
- 사용자: 인증이 없는 동안은 mock 사용자 id 1 로 고정한다. "현재 사용자 기준" 필드(팔로우 여부, 멤버십)는 이 사용자로 계산한다.
- 시간: 날짜로 거르는 파라미터(YYYY-MM-DD)는 한국 시간 기준으로 비교한다. `lib/utils.ts` 의 `formatDateTime` 과 같은 방식.
- 테스트: `vi.stubEnv("NEXT_PUBLIC_USE_MOCK", "true")` 로 mock 모드에서 호출 함수를 한 번씩 부른다. 쓰기는 다음 읽기에 반영되는지, 오류는 상태 코드가 맞는지 본다.
- 시나리오 주석: mock 조회 · 쓰기 함수마다 바로 위에 정상 · 예외 시나리오를 아래 형식으로 컴팩트하게 적는다. 테스트는 이 주석의 case 를 하나씩 확인한다.

```ts
/**
 * POST /api/{resources}/:id/{action}
 * case 1: 인자 { id: 있는 모임, joinPolicy: OPEN }
 *   동작: 멤버십 MEMBER 추가, memberCount +1
 *   정상 응답: { membership: "MEMBER", memberCount: n+1 }
 * case 2: 인자 { id: 있는 모임, 이미 멤버 }
 *   동작: 아무것도 바꾸지 않음
 *   정상 응답: { membership: 기존 값, memberCount: n }
 * case 3: 인자 { id: 없는 모임 }
 *   예외: 404 "모임을 찾을 수 없어요"
 * case 4: 인자 { id: 있는 모임, joinPolicy: REVIEW }
 *   예외: 400 "심사제 모임은 지원서를 내야 해요"
 */
export function joinGroup(id: number) { ... }
```

### 권장 동작 순서

1. 문서 읽기
   `api.md` 의 상태가 "검수 완료" 인지, "결정 필요" 가 비어 있는지 본다. 아니면 사용자에게 알리고 멈춘다. "새로 필요" 항목과 요청 · 응답 표를 뽑는다. `data.md` 의 API 데이터 타입과 대조한다.

2. 있는 것 확인
   `src/types/api/`, `src/lib/*-data.ts`, `handlers.ts`, `src/lib/*-api.ts` 를 읽고, 만들 것과 보탤 것을 나눈다.

3. 파일 쓰기
   타입 → mock 데이터와 조회 함수 → handlers 항목 → 호출 함수 → 테스트 순서.

4. 검증
   `npm run typecheck && npm run lint && npm run test:run`.

5. 문서 갱신
   `api.md` 의 해당 항목 상태를 "있음 ({함수})" 으로 바꾸고 머리 상태를 "구현됨" 으로 바꾼다.

### 종료 조건

- [ ] api.md 의 "새로 필요" 가 전부 "있음" 이 되었다
- [ ] 엔드포인트마다 타입 · handlers 항목 · 호출 함수 · 테스트가 있다
- [ ] 검증 명령을 통과했다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- api.md 의 상태가 "검수 완료" 가 아니거나 "결정 필요" 가 비어 있지 않다
- api.md 의 응답 필드가 data.md 의 타입과 다르다
- 이미 있는 파일과 겹치는데 어느 쪽이 맞는지 알 수 없다
