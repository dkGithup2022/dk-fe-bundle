# fe-page-api 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 홈(/) 페이지 실행, 2026-09-23. 이 홈은 뒤에 디자인 랩을 거쳐 다시 바뀌었지만, 체인을 README 의 순서대로 처음 끝까지 돈 사례라 이것을 싣는다.

## 입력

- 홈 화면 이미지
- `data.md`, `components.md`
- 이미 있는 API (mock 등록 목록, 호출 함수)

## 사람이 정한 것

- 참석 확인은 attended 만 저장한다. s06 [다음에 할게요] 는 시트를 닫기만 하고 저장하지 않는다. 별도 닫기 API 없음 (사용자 결정)
- 알림 수는 알림 목록 API (`read=false&size=1`) 의 totalElements 로 센다. 요약 API 없음 (사용자 결정)

## 산출물

### 목록

출처: `docs/pages/home/api.md`

```markdown
## 목록

| # | 종류 | 이름 | 트리거 | 상태 |
|---|---|---|---|---|
| 1 | 유즈케이스 | 피드 읽기 (게시글 + 후기 섞어 최신순) | 진입, 카테고리 탭 | 있음 (lib/feed-api.ts fetchFeed) |
| 2 | 도메인 CRUD | 읽지 않은 알림 수 | 진입 | 있음 (lib/notification-api.ts fetchNotifications) |
| 3 | 유즈케이스 | 참석 확인 대상 읽기 | 진입 | 있음 (lib/attendance-check-api.ts fetchAttendanceCheckPrompt) |
| 4 | 유즈케이스 | 참석 확인 응답 (네 / X) | 배너 [네], [X] | 있음 (lib/attendance-check-api.ts answerAttendanceCheck) |
| 6 | 도메인 CRUD | 내 참석 이벤트 목록 | s11 진입, 더 보기 | 있음 (lib/me-api.ts fetchMyEvents) |
| 7 | 도메인 CRUD | 내 모임 목록 | s12 진입 | 있음 (lib/me-api.ts fetchMyGroups) |
| 8 | 페이지 동작 | 카테고리 탭 · 지역 탭 | 탭 | API 아님. 카테고리는 1번 파라미터, 지역은 UI 만 |
| 9 | 페이지 동작 | FAB → 글쓰기 시트 열기, 하나 고르기, s11 · s12 라디오 | 누름 | API 아님 |
| 10 | 페이지 동작 | 참석 확인 시트 열기 · 닫기 | [네] 뒤 열림, [다음에 할게요] 로 닫힘 | API 아님. 닫기는 저장하지 않는다 (결정) |
| 11 | 페이지 동작 | 이동: 검색, 알림, 피드 카드, [후기 남기러 가기], [다음] | 누름 | API 아님. 라우트 없음 → 이번엔 동작 빼 둠 |
| 12 | 페이지 동작 | 하단 탭 → /groups | 탭 | API 아님. 라우트 있음 → 연결 |
```

### 유즈케이스 한 건

출처: `docs/pages/home/api.md`

```markdown
## 3. 유즈케이스

### GET /api/feed — 피드 읽기
- 목적: 홈 첫 화면에 게시글과 후기를 섞어 보여준다 (s01 D)
- 동작: 발행된 모임 게시글과 공개 후기를 하나의 목록으로 합쳐 최신순(게시글 publishedAt · 후기 createdAt)으로 돌려준다. 항목마다 종류(type)가 붙고, 게시글은 모임 이름과 본문 앞부분(summary), 후기는 작성자 닉네임 · 아바타와 이벤트 제목을 같이 준다. 한 도메인의 목록이 아니라 유즈케이스
- 인증: 불필요
- 상태: 있음 (구현됨)

요청

| 위치 | 이름 | 타입 | 필수 | 뜻 · 근거 |
|---|---|---|---|---|
| query | region | string | 아니오 | s01 C. 지금은 안 보냄 (UI 만) |
| query | categoryGroup | CategoryGroup | 아니오 | s01 C 탭. 게시글은 모임의 카테고리, 후기는 이벤트가 속한 모임의 카테고리로 거른다 |
| query | page, size | number | 아니오 | 기본 0, 20 |

응답 `FeedListResponse` = PaginatedResponse<FeedItem>

| 필드 | 타입 | 뜻 · 화면 어디에 |
|---|---|---|
| type | "ARTICLE" \| "REVIEW" | 카드 종류 |
| (ARTICLE) id, title, summary, coverImageUrl, publishedAt, likeCount, groupId, groupName | | s01 게시글 카드 |
| (REVIEW) id, body, photoUrl, authorNickname, authorAvatarUrl, eventId, eventTitle, createdAt | | s01 후기 카드 |

### GET /api/me/attendance-check — 참석 확인 대상 읽기
- 목적: 홈 상단 배너에 "참석 확인" 할 이벤트를 보인다 (s01 B)
- 동작: 현재 사용자가 참석 신청한 이벤트 중 이미 지났고 아직 참석 확인 응답이 없는 것 가운데 가장 최근 하나를 돌려준다. 없으면 null. 참석 확인 응답(feature 데이터)을 보고 계산하므로 유즈케이스
- 인증: 필요
- 상태: 있음 (구현됨)

요청: 없음

…(이하 생략)
```
