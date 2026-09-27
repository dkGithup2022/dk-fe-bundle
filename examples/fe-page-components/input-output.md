# fe-page-components 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 홈(/) 페이지 실행, 2026-09-23. 이 홈은 뒤에 디자인 랩을 거쳐 다시 바뀌었지만, 체인을 README 의 순서대로 처음 끝까지 돈 사례라 이것을 싣는다.

## 입력

- 홈 화면 이미지
- `src/components/` 의 기존 컴포넌트
- `docs/pages/home/data.md`

## 사람이 정한 것

- "참석 확인" 코럴 라벨: Label 을 고치지 않고 색을 고를 수 있는 공용 ToneLabel 을 새로 둔다 (사용자 결정)
- s10 체크 항목(체크 + 제목 + 설명): Checkbox 를 고치지 않고 페이지 컴포넌트 WriteKindOption 으로 만든다 (사용자 결정)

## 산출물

### 영역

출처: `docs/pages/home/components.md`

```markdown
## 영역

- A 상단 바: 왼쪽 글자 로고 "Peeple"(민트 강, 굵게), 오른쪽 "검색" · "알림" 글자 + 읽지 않은 알림이 있으면 코럴 점. 뒤로 버튼 없음
- B 참석 확인 배너: 코럴 라벨 "참석 확인", 이벤트 제목 두 줄, 오른쪽 원형 민트 버튼 [네]. 결정대로 X 추가
- B' 참석 확인 시트(s06): 제목 "후기를 남기면 다음 참여를 독려할 수 있어요", 설명 한 줄, 이벤트 줄(강조 날짜 블록 · 제목 · "오후 7:15 · 자동으로 첨부됩니다"), [후기 남기러 가기] 민트 왕버튼, [다음에 할게요] 글자 버튼
- C 필터 줄: "Seoul, KR"(밑줄 선택) · 카테고리들. 밑줄 탭 모양
- D 피드: 게시글 카드(표지 16:9 · 라벨 "이야기 · ChillPal 주최" · 제목 19/700 두 줄 · 요약 두 줄 · "날짜 시각" 과 "♡ 42") / 후기 카드(아바타 · 닉네임 · "· 후기" · 본문 세 줄 · 이벤트 제목 · 오른쪽 84 썸네일). 카드 사이 구분선
- E FAB(+) 와 글쓰기 시트: s10 체크 항목 두 개(제목 + 설명) + [다음](고르기 전 비활성). s11 라벨 "어떤 이벤트인가요? · 최신순" + 행(날짜 블록 · 제목 · "ChillPal · 오후 7:15" · 라디오) + "더 보기". s12 라벨 "어느 모임 이름으로 쓸까요?" + 행(썸네일 · 이름 · "운영자 · 멤버 1,241" · 라디오)
- F 하단 탭
```

### 추가 컴포넌트

출처: `docs/pages/home/components.md`

```markdown
## 추가 컴포넌트

| 이름 | 위치 | props | 안에 쓰는 공용 컴포넌트 | 근거 |
|---|---|---|---|---|
| FeedArticleCard | src/components/feed/FeedArticleCard.tsx | item: FeedArticleItem, onClick?: () => void | ui/Thumb(cover), ui/Label | s01 D. 피드에서 반복. 디자인 시트는 "홈 피드 카드 — 공용으로 두지 않을 것" 이라 ui/ 가 아닌 도메인 폴더 |
| FeedReviewCard | src/components/feed/FeedReviewCard.tsx | item: FeedReviewItem, onClick?: () => void | ui/Thumb(avatar, square) | s01 D. 위와 같음. 게시글 카드와 위계가 달라 별도 |

페이지 본문에 두는 것 (한 번만 나옴): A 상단 바 (`src/app/HomeTopBar.tsx`), B 참석 확인 배너 (`src/app/AttendanceBanner.tsx`), B' 시트 내용, E 글쓰기 시트 세 단계 (`src/app/WriteSheet.tsx`), s10 의 체크 항목 한 줄(체크 + 제목 + 설명, `src/app/WriteKindOption.tsx`. 사용자 결정). 파일로 나누는 건 페이지 안에서만.
```
