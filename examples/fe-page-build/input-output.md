# fe-page-build 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 홈(/) 페이지 실행, 2026-09-23. 이 홈은 뒤에 디자인 랩을 거쳐 다시 바뀌었지만, 체인을 README 의 순서대로 처음 끝까지 돈 사례라 이것을 싣는다.

## 입력

- 상태가 "허가" 인 `docs/pages/home/plan.md`와 나머지 문서
- 홈 화면 이미지 (마지막 대조용)

## 사람이 정한 것

- 실제 라우트를 화면과 대조하고 끝냈다

## 산출물

### 제작 기록

출처: `docs/pages/home/plan.md`

```markdown
## 6. 제작 기록

- 만든 파일: src/app/{page,HomeClient,loading,HomeTopBar,AttendanceBanner,AttendanceSheet,WriteSheet,WriteKindOption}.tsx, home.types.ts · components/feed/{FeedArticleCard,FeedReviewCard} (+ .types.ts)
- 계획과 달라진 것: 글쓰기 시트의 상태(writeSheet, myEvents, myGroups)를 HomeClient 가 아니라 WriteSheet.tsx 안에 두었다. 시트 밖에서 쓰지 않아서. 나머지 가정값은 그대로
- 제작 중 고친 것: 첫 진입 요청을 loadFeed 로 부르면 lint(set-state-in-effect)에 걸려, effect 안에서는 fetchFeed 를 then 으로 직접 받음
- 화면과 다른 곳: 배너에 [네] 를 누르면 배너가 사라지고 시트가 뜬다 (디자인 노트 "화면이 바뀌지 않고 시트만" 과 거의 같음). 탭 문구는 결정대로 CategoryGroup
- 와이어프레임 라우트 삭제함
```
