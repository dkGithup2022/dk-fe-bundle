# fe-apply-theme 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 실행일 2026-09-21.

## 입력

- 디자인 export 의 팔레트 · 타이포 표 (색 14개, 타이포 8단계, 규칙 4개)
- 템플릿 `globals.css` 의 토큰 이름 목록

## 사람이 정한 것

- 기본 토큰에 자리가 없는 색 4개(코럴 약 · 강, 민트 위 글자색 등)를 확장 토큰으로 추가한다
- 디자인에 다크 값이 없어 라이트 전용으로 간다
- 보조 글자색과 비활성 글자색은 구분만 되면 된다

## 산출물

### 테마 파일

출처: `src/styles/themes/peeple.css`

```css
/* ================================================================
   Peeple 테마
   출처: working_skills/project_ref/export-system 팔레트 표 8a.
   디자인에 다크 값이 없어 라이트 전용이다.
   "유도" 표시는 디자인 표에 없어 가까운 값에서 만든 것.
   ================================================================ */

:root {
  --bg-primary: #fcfeff;    /* 배경 — 화면 전체 바탕, 이 한 겹만 */
  --bg-secondary: #f3f8f6;  /* 유도 — 배경과 비활성 바탕 사이 */
  --bg-tertiary: #e7f0ee;   /* 비활성 버튼 바탕 */
  --bg-elevated: #ffffff;   /* 면 — 시트·모달 */
  --bg-surface: #ffffff;    /* 면 — 입력칸 안쪽 */

  --accent-primary: #67dcc3;      /* 민트 중 — 주 버튼 칠, 탭 밑줄. 위 글자는 --text-on-accent */
  --accent-secondary: #077a6a;    /* 민트 강 — 흰 글자 버튼, 활성 탭 아이콘 */
  --accent-muted: #c4ede3;        /* 민트 경계 — 아웃라인 버튼 테두리 */
  --accent-glow: #e2f7f1;         /* 민트 약 — 칩 바탕 */
  --accent-glow-strong: #cff2e8;  /* 유도 — 선택된 항목 바탕, 민트 약과 중 사이 */
  --accent-warm: #ff6b4a;         /* 코럴 중 — 알림 점, 아이콘. 글자 올리지 않음 */

  --text-primary: #13322c;    /* 먹 — 제목·본문 */
  --text-secondary: #5f7570;  /* 보조 글자 — 날짜·수치·설명, 입력칸 안내 */
  --text-tertiary: #4a5f5a;   /* 비활성 글자 — 바탕 --bg-tertiary */
  --text-accent: #077a6a;     /* 민트 강 — 라벨 글자 */

  --border-subtle: #e7f0ee;   /* 구분선 — 섹션 사이 1px */
  --border-default: #e7f0ee;  /* 구분선 — 입력칸 테두리 */
  --border-hover: #c4ede3;    /* 민트 경계 — 포커스 테두리 */

  --shadow-sm: none;  /* 디자인 규칙: 카드 테두리·그림자를 쓰지 않는다 */
  --shadow-md: none;
  --shadow-lg: none;

  --font-body: Pretendard, system-ui, -apple-system, sans-serif;

  /* 타이포 8단계 — 디자인 표 "타이포" */
  --fs-title-l: 19px;   /* 글 제목 L — 피드·뷰어 제목. 700 */
  --fs-title-m: 18px;   /* 글 제목 M — 두 번째 글, 화면 제목. 700 */
  --fs-title-s: 15px;   /* 제목 S — 모임 이름, 섹션 제목. 700 */
…(이하 생략)
```
