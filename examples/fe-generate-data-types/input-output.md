# fe-generate-data-types 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 검수된 스펙 v2 를 그대로 옮겼다.

## 입력

- `docs/data-spec-v2.md` (결정 필요가 비어 있는 상태)

## 사람이 정한 것

- 없음. 판단하지 않는 스킬이다

## 산출물

### 도메인 타입 예시

출처: `src/types/data/domain/group.ts`

```ts
import type { Place } from "./place";

/**
 * 모임 (도메인) — 이름 · 지역 · 소개 · 카테고리 · 위치 · 가입 방식을 가진 단위. 이벤트와 모임 게시글을 발행
 * 근거 화면: s04-group.png, s02-groups.png, s13-create1.png, s14-create2.png, s15-create3.png
 */
export interface Group {
  id: number;
  name: string;                   // s04 "ChillPal"
  region: string;                 // s04 "Seoul, KR"
  memberCount: number;            // s04 "멤버 1,241"
  joinPolicy: JoinPolicy;         // s04 "자유 가입" 배지, s15 "지원서 받기" 토글
  isRegular: boolean;             // 정기 모임 여부. 매주인지 월 2회인지는 두지 않는다
  coverImageUrl: string | null;   // s04 커버, s13 "대표 사진"
  tagline: string;                // s13 "한 줄 소개"
  description: string;            // s04 소개글, s13 "소개글"
  primaryCategoryId: string;      // s14 하위 카테고리 중 대표 하나
  categoryIds: string[];          // s14 소속 카테고리. 대표를 포함한다
  location: Place | null;         // s14 위치 검색 · 지도 핀 · 주소
  applicationQuestions: string[]; // s15 "질문 1", "질문 2". 심사제가 아니면 빈 배열
}

/** 가입 방식 (enum) — s04-group.png, s15-create3.png */
export type JoinPolicy = "OPEN" | "REVIEW";
export const JOIN_POLICY_LABEL: Record<JoinPolicy, string> = {
  OPEN: "자유 가입",
  REVIEW: "심사제",
};

/** 상위 카테고리 (enum) — s14-create2.png */
export type CategoryGroup = "HOBBY" | "STUDY" | "EXERCISE" | "CAREER";
export const CATEGORY_GROUP_LABEL: Record<CategoryGroup, string> = {
  HOBBY: "취미",
  STUDY: "공부",
  EXERCISE: "운동",
  CAREER: "커리어",
};

/** 하위 카테고리 — s14-create2.png. 화면에 자리표시자("취미 A")뿐이라 실제 목록은 BE 또는 기획에서 온다 */
export interface Category {
  id: string;
  group: CategoryGroup;
  label: string;
}

/** 하위 카테고리 id. 모임의 categoryIds 와 유저의 interests 가 같은 값 공간을 쓴다 (docs/pages/group-apply/data.md 결정) */
export type CategoryId = Category["id"];
```
