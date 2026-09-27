# fe-page-plan 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 홈(/) 페이지 실행, 2026-09-23. 이 홈은 뒤에 디자인 랩을 거쳐 다시 바뀌었지만, 체인을 README 의 순서대로 처음 끝까지 돈 사례라 이것을 싣는다.

## 입력

- `data.md`, `components.md`, `api.md`
- 홈 화면 이미지
- `src/` 의 현재 상태

## 사람이 정한 것

- 충돌 검사와 정하지 않은 UI 세부를 보고 상태를 "허가" 로 바꿨다

## 산출물

### 충돌 검사

출처: `docs/pages/home/plan.md`

```markdown
## 1. 충돌 검사

| # | 어디와 어디 | 내용 | 제안 |
|---|---|---|---|
| 1 | components.md FeedArticleCard · FeedReviewCard ↔ src/components | 두 컴포넌트가 아직 없음 | 예정. 제작 단계에서 `components/feed/` 에 만든다 |
| 1b | api.md 피드 응답 ↔ components.md 카드 props | FeedArticleItem · FeedReviewItem 이 카드가 그리는 필드를 전부 가짐. 날짜 문구는 formatDateTime(있음) | 충돌 없음 |
| 2 | data.md HomeFilter.categoryGroup(null = 전체) ↔ ui/Tabs value(string) | "Seoul, KR" 탭이 전체를 뜻하므로 값 "REGION" 을 전체로 쓴다 | 페이지에서 변환. 문서 수정 없음 |
| 3 | 문서 ↔ src/app | 템플릿의 HomeClient.tsx (health 확인용) 와 loading.tsx 가 이미 있음 | 덮어쓴다. page.tsx 는 metadata 만 바꾼다 |
| 4 | 화면 ↔ 결정 | 배너 X 버튼, 탭 문구(카테고리 4개) 는 디자인에 없음 | 결정대로 만들고 디자인에 알린다 |
| 5 | api.md 11번 이동 5곳 ↔ components.md 카드 onClick | 라우트 없음 | 카드에 onClick 을 넘기지 않는다. 검색 · 알림 글자도 동작 없음 |
| 6 | data.md 참석 확인 응답(reviewPromptDismissed) ↔ api.md 결정 | 닫기를 저장하지 않기로 함 | 필드는 타입에 있지만 페이지는 쓰지 않는다 |
```

### 정하지 않은 UI 세부

출처: `docs/pages/home/plan.md`

```markdown
## 3. 정하지 않은 UI 세부

| 영역 | 항목 | 가정값 | 근거 · 이유 |
|---|---|---|---|
| A | 높이 · 로고 크기 | 48px · text-title-m 민트 강 | s01 눈대중 |
| A | 검색 · 알림 글자 | text-desc, 보조 글자색. 점은 Badge dot 를 글자 오른쪽 위 | s01 |
| B | 배너 패딩 | 세로 8, 좌우 gutter | 눈대중 |
| B | X 위치 · 모양 | [네] 왼쪽에 작은 X 아이콘 버튼 (32) | 사용자 결정 |
| B | [네] 버튼 | 원형 44, 민트 중 | s01 |
| B | prompt 없을 때 | 배너 영역 자체를 그리지 않음 | 화면에 없음 |
| C | 탭 | size sm. "Seoul, KR" 은 선택 모양만 (동작 없음) | 결정 |
| D | 카드 세로 패딩 · 사이 구분선 | 16px · inset 0 | 눈대중 |
| D | 게시글 제목 · 요약 | text-title-l 두 줄 · text-body 보조색 두 줄 | s01 |
| D | 후기 카드 | 본문 세 줄, 썸네일 square 84 오른쪽 | s01 |
| D | 더 불러오기 | 50개씩. 스크롤 끝에서 다음 페이지 자동 로딩. 로딩 중 표시는 목록 아래 작은 스피너 | 사용자 결정 |
| D | 빈 상태 | EmptyState "아직 글이 없어요" | 화면에 없음 |
| E | FAB 위치 | 하단 탭 위 24px, 오른쪽 gutter. 컨테이너 폭 기준 | s01 |
| E | 시트 목록 | s11 은 3개 + 더 보기, s12 는 전부 | s11 · s12 |
| E | [다음] | kind 와 eventId/groupId 가 다 있어야 활성 | s10 비활성 모양 |
| 전체 | 로딩 | loading.tsx (상단 바 · 배너 · 탭 · 카드 2) | |
| 전체 | 에러 | showErrorToast + 빈 피드 | |
```
