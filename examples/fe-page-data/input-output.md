# fe-page-data 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 홈(/) 페이지 실행, 2026-09-23. 이 홈은 뒤에 디자인 랩을 거쳐 다시 바뀌었지만, 체인을 README 의 순서대로 처음 끝까지 돈 사례라 이것을 싣는다.

## 입력

- 홈 화면 이미지 (s01 홈, s06 참석 확인 시트, s10 · s11 · s12 글쓰기 시트)
- 데이터 스펙과 `src/types/data/`

## 사람이 정한 것

- C 첫 칩 "Seoul, KR": UI 만 둔다. 누르는 동작 없음. 지역 선택지가 정해지면 필터로 (사용자 결정)
- D 피드 정렬: 최신순 하나. 게시글 publishedAt · 후기 createdAt 을 같은 축으로 (사용자 결정)
- B 배너: "네" 옆에 X 를 둔다. X 는 참석 안 함(attended = false). 디자인에 알릴 것 (사용자 결정)
- s10 글쓰기 시트: 두 항목 중 하나만 고른다. 하나를 고르면 다른 쪽은 풀린다. 모양은 체크박스 유지 (사용자 결정)

## 산출물

### 공용 데이터

출처: `docs/pages/home/data.md`

```markdown
## 공용 데이터 (스펙 그대로 사용)

| 항목 | 위치 | 쓰는 필드 | 쓰는 영역 |
|---|---|---|---|
| 모임 게시글 | src/types/data/domain/group-article.ts | id, groupId, kind, coverImageUrl, title, body(앞부분), publishedAt, likeCount | D 게시글 카드 |
| 모임 후기 | src/types/data/domain/review.ts | id, eventId, authorId, body, photoUrls(첫 장), createdAt | D 후기 카드 |
| 유저 | src/types/data/domain/user.ts | nickname, avatarUrl | D 후기 카드 작성자 |
| 이벤트 | src/types/data/domain/event.ts | id, groupId, title, startsAt | B 배너 · s06, D 후기 카드의 이벤트 제목, s11 목록 |
| 모임 | src/types/data/domain/group.ts | id, name, coverImageUrl, memberCount | D 라벨 "ChillPal 주최", s11 "ChillPal", s12 목록 |
| 참석 | src/types/data/domain/attendance.ts | userId, eventId | s11 "최근 참석 이벤트", B 배너 대상 |
| 멤버십 | src/types/data/domain/membership.ts (MemberRole, MEMBER_ROLE_LABEL) | role | s12 "운영자" · "멤버" |
| 알림 | src/types/data/domain/notification.ts | isRead | A 알림 점 |
| 상위 카테고리 | src/types/data/domain/group.ts (CategoryGroup, CATEGORY_GROUP_LABEL) | 값 4개와 라벨 | C 칩 |
| 게시글 종류 | src/types/data/domain/group-article.ts (GroupArticleKind, GROUP_ARTICLE_KIND_LABEL) | 라벨 "이야기" | D 라벨 |
| 참석 확인 응답 | src/types/data/feature/attendance-check.ts | userId, eventId, attended, reviewPromptDismissed | B 배너 · s06 시트 |
| 글쓰기 종류 | src/types/data/feature/write.ts (WriteKind, WRITE_KIND_LABEL) | REVIEW · GROUP_ARTICLE | s10 체크 |
```

### API 데이터

출처: `docs/pages/home/data.md`

```markdown
## API 데이터

| 이름 | 위치 | 필드 | 용도 |
|---|---|---|---|
| FeedListParams | src/types/api/feed.ts | region?: string, categoryGroup?: CategoryGroup, page, size | C 조건으로 D 요청 |
| FeedArticleItem | src/types/api/feed.ts | type: "ARTICLE", id, title, summary: string (body 앞부분. 서버가 잘라 줌), coverImageUrl, publishedAt, likeCount, groupId, groupName | D 게시글 카드 |
| FeedReviewItem | src/types/api/feed.ts | type: "REVIEW", id, body, photoUrl: string \| null, authorNickname, authorAvatarUrl, eventId, eventTitle, createdAt | D 후기 카드 |
| FeedItem | src/types/api/feed.ts | FeedArticleItem \| FeedReviewItem | D 한 장 |
| FeedListResponse | src/types/api/feed.ts | PaginatedResponse<FeedItem> | D |
| AttendanceCheckPrompt | src/types/api/attendance-check.ts | eventId, eventTitle, startsAt | B 배너와 s06 시트에 보일 이벤트. 없으면 null |
| AttendanceCheckAnswerRequest | src/types/api/attendance-check.ts | eventId, attended: boolean | B "네" (true), X (false) |
| MyAttendedEventListItem | src/types/api/me.ts | id, title, groupName, startsAt | s11 목록 |
| MyAttendedEventListResponse | src/types/api/me.ts | PaginatedResponse<MyAttendedEventListItem> | s11 더 보기 |
| MyGroupListItem | src/types/api/me.ts | id, name, coverImageUrl, memberCount, role: MemberRole | s12 목록 |
| MyGroupListResponse | src/types/api/me.ts | PaginatedResponse<MyGroupListItem> | s12 |
| NotificationListResponse | src/types/api/notification.ts | PaginatedResponse<Notification> | A 알림 점 (read=false 로 불러 totalElements 만 씀) |
```
