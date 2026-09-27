---
name: fe-page-build
description: 허가된 docs/pages/{라우트 이름}/plan.md 를 읽어 페이지를 만든다. 페이지 파일, 도메인 컴포넌트, 유틸을 만들고 검증한 뒤 실제 라우트를 화면과 대조한다. 계획서에 없는 것은 만들지 않는다.
argument-hint: "[라우트 이름]"
---

# 페이지 제작

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

허가된 계획서를 그대로 코드로 옮기는 것이 목적입니다. 판단은 계획서에 이미 있습니다. 계획서에 없는 컴포넌트 · 상태 · 동작을 더하지 않고, 계획서의 가정값을 임의로 바꾸지 않습니다. 끝나면 실제 라우트를 띄워 디자인과 대조합니다.

### 입력 & 출력

입력:
- `docs/pages/{라우트 이름}/plan.md`. 상태가 "허가" 여야 한다.
- `docs/pages/{라우트 이름}/data.md`, `components.md`, `api.md`. 계획서가 가리키는 세부.
- 페이지 디자인 이미지. 마지막 대조에 쓴다.
- `src/` 의 현재 상태. 계획서가 "있음" 이라고 한 것이 실제로 있는지 본다.

출력:
- `src/app/{route}/page.tsx` — 서버. `metadata` 또는 동적 라우트면 `generateMetadata`. `<Suspense>` 로 Client 를 감싼다.
- `src/app/{route}/{Name}Client.tsx` — 본문. 상태와 이벤트.
- `src/app/{route}/loading.tsx` — 스켈레톤. 본문과 같은 구조.
- `src/app/{route}/{name}.types.ts` — 페이지 전용 타입 (data.md).
- `src/app/{route}/{Part}.tsx` — 계획서가 페이지 전용으로 나눈 부분 (헤더, 탭 내용, 검색 줄).
- `src/components/{domain}/{Name}.tsx` + `.types.ts` — components.md 의 추가 컴포넌트.
- `src/lib/utils.ts` 등 — 계획서 "판단" 절이 요구한 유틸.
- 계획서 6절 "제작 기록".

```tsx
// src/app/{route}/page.tsx — 정적 라우트
import { Suspense } from "react";
import type { Metadata } from "next";
import { {Name}Client } from "./{Name}Client";

export const metadata: Metadata = { title: "{제목}", description: "{설명}" };

export default function Page() {
  return (
    <Suspense>
      <{Name}Client />
    </Suspense>
  );
}
```

```tsx
// src/app/{route}/[id]/page.tsx — 동적 라우트
export async function generateMetadata({ params }: PageProps<"/{route}/[id]">): Promise<Metadata> {
  const { id } = await params;
  try {
    const item = await fetch{Name}Detail(Number(id));
    return { title: item.name };
  } catch {
    return { title: "{기본 제목}" };
  }
}

export default async function Page({ params }: PageProps<"/{route}/[id]">) {
  const { id } = await params;
  const itemId = Number(id);
  if (!Number.isInteger(itemId) || itemId <= 0) notFound();
  return (
    <Suspense>
      <{Name}Client id={itemId} />
    </Suspense>
  );
}
```


### 작성 규칙

- 상태는 계획서의 상태 표 그대로. 전부 `useState`. 계획서가 store 로 표시한 것만 `store/`.
- 동작은 계획서의 동작 표 그대로. "없음 (동작 빼 둠)" 인 것은 핸들러도 `href` 도 만들지 않는다. 라우트가 생길 때 연결한다.
- API 는 `lib/{domain}-api.ts` 의 함수만 부른다. 경로 문자열을 페이지에 쓰지 않는다.
- 도메인 컴포넌트는 fe-apply-components 의 작성 규칙을 따른다. 도메인 타입 props 는 허용.
- 공용(`ui/`) 컴포넌트는 만들지도 고치지도 않는다. 필요하면 멈추고 fe-apply-components 로 돌린다.
- effect 본문에서 동기 `setState` 를 부르지 않는다 (lint react-hooks/set-state-in-effect). 첫 진입 요청은 `then` 으로 받고, 첫 로딩 표시는 초기 상태로 둔다. 외부 구독 콜백이나 DOM 측정처럼 정말 필요하면 그 줄에 이유를 적고 끈다.
- 렌더 중에 `Date.now()` 같은 불순 함수를 부르지 않는다 (lint react-hooks/purity). 기준 시각은 응답이 올 때나 이벤트 핸들러에서 상태에 넣는다.
- `line-clamp-*` 는 flex 아이템에 붙이면 풀린다. 블록 요소로 한 번 감싸거나 `block` 으로 쌓는다.
- 전체 컨테이너와 각 하위 컴포넌트와 구성 요소는 고정 크기보다는 flex 를 사용하는 것이 권장됩니다.
- 크기를 직접 선언해야 한다면 숫자를 넣기 보다는 tailwind 변수에 제안된 크기 숫자가 있는지 확인하세요.

### 권장 동작 순서

1. 계획서 읽기
   상태가 "허가" 인지 본다. 아니면 멈춘다. 파일 목록 · 컴포넌트 트리 · 상태 표 · 동작 표 · 가정값 · 판단을 뽑는다. `src/` 에서 "있음" 인 것이 실제로 있는지 본다.

2. 유틸 · 도메인 컴포넌트
   계획서 "판단" 의 유틸을 먼저 만들고 테스트를 붙인다. components.md 의 추가 컴포넌트를 만든다.

3. 페이지 파일
   타입 → 페이지 전용 부분 → Client → page → loading 순서. 계획서 3절의 가정값을 그대로 쓴다.

4. 정리
   와이어프레임 라우트(`src/app/dev/wireframe/{라우트 이름}/`)를 지운다.

5. 검증
   `npm run typecheck && npm run lint && npm run test:run`. lint 오류는 규칙 이름을 읽고 고친다.

6. 화면 대조
   개발 서버(mock)로 실제 라우트를 띄워 스크린샷을 찍고 디자인 이미지와 나란히 본다. 탭 · 시트처럼 상태가 있는 것은 눌러서 각각 찍는다. 다른 곳은 고치되, 계획서 가정값을 바꿔야 하면 계획서에도 적는다. 없는 id 같은 오류 경로도 한 번 연다.

7. 기록
   계획서 상태를 "제작 완료" 로 바꾸고 6절 "제작 기록" 에 만든 파일, 계획과 달라진 것, 제작 중 고친 것, 스크린샷 경로를 적는다. `docs/pages/ORDER.md` 같은 순서 문서가 있으면 상태를 바꾼다.

### 종료 조건

- [ ] 계획서의 파일 목록과 컴포넌트 트리가 전부 코드로 있다
- [ ] 동작 표의 모든 동작이 구현되었거나 "동작 빼 둠" 으로 남아 있다
- [ ] 검증 명령을 통과했다
- [ ] 실제 라우트 스크린샷이 디자인과 대조되었고 계획서 6절에 기록되었다
- [ ] 와이어프레임 라우트가 지워졌다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- 계획서 상태가 "허가" 가 아니다
- 계획서가 "있음" 이라고 한 API · 컴포넌트가 코드에 없다
- 만들다 보니 공용 컴포넌트를 고쳐야 한다
- 화면 대조에서 계획서 가정값으로는 디자인과 맞출 수 없다
