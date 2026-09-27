---
name: fe-apply-theme
description: 디자인 핸드오프의 팔레트·타이포 표를 읽어 테마 리소스 파일 src/styles/themes/{name}.css 를 새로 만들고, Next.js 앱이 그 파일을 읽게 한다. 토큰 이름은 템플릿의 것을 그대로 쓴다.
argument-hint: "[디자인 export 폴더] [테마 이름]"
---

# 색 테마 만들기

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

디자인 핸드오프의 색 표를 템플릿 토큰 이름을 가진 리소스 파일로 만들고, 앱이 그 파일을 읽게 하는 것이 목적입니다. 이후 컴포넌트와 페이지는 이 토큰만 씁니다.

### 입력 & 출력

입력:
- 사용자가 지정한 디자인 export 폴더. `*.dc.html` 중 팔레트·타이포 표가 있는 파일과 `README.md`. `support.js` 는 렌더 런타임이므로 읽지 않는다.
- `src/app/globals.css` 의 토큰 이름 목록과 현재 import 중인 테마.
- `src/styles/themes/` 에 이미 있는 파일.
- 대화에서 사용자가 정한 사항 (확정안 번호, 다크 모드 여부 등).

출력:
- `src/styles/themes/{name}.css`: 토큰 값. 토큰마다 디자인 표의 이름을 주석으로 남긴다.
- `src/app/globals.css`: 위 파일을 `@import` 한다. (연결하기로 했을 때만)
- `src/app/layout.tsx`: 글꼴 불러오기. (디자인이 글꼴을 정했을 때만)

```css
/* src/styles/themes/peeple.css — 출처: export-system 팔레트 표 8a. 라이트 전용 */
:root {
  --bg-primary: #fcfeff;     /* 배경 */
  --bg-secondary: #f3f8f6;   /* 유도: 배경보다 한 단계 어둡게 */
  --bg-surface: #ffffff;     /* 면 */
  --accent-primary: #67dcc3; /* 민트 중 */
  --text-primary: #13322c;   /* 먹 */
  --border-subtle: #e7f0ee;  /* 구분선 */
  --shadow-sm: none;         /* 디자인 규칙: 그림자 없음 */
  /* ... 나머지 토큰 */
  --font-body: Pretendard, system-ui, sans-serif;
  color-scheme: light;
}

/* 프로젝트 확장 토큰 — 기본 토큰에 자리가 없어 사용자 확인 후 추가한 것 */
:root {
  --text-on-accent: #0a3c33; /* 민트 중 위 글자 */
  --signal-soft: #ffe7e0;    /* 코럴 약 */
  --signal-strong: #c22f16;  /* 코럴 강 */
}
```

```css
/* src/app/globals.css — 맨 위 */
@import "tailwindcss";
@import "../styles/themes/peeple.css";
```

```tsx
// src/app/layout.tsx — next/font/google 에 없는 글꼴은 <link> 로, 있으면 next/font/google 로
<head>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css" />
</head>
```

### 권장 동작 순서

1. 작업 방식 확인
   `src/styles/themes/` 에 파일이 있거나 `globals.css` 가 이미 테마를 import 하고 있으면 사용자에게 묻는다. 지금 테마를 새 파일로 바꿀지, 파일만 만들고 연결은 하지 않을지. 둘 다 없으면 만들고 연결한다.

2. 토큰 이름 확인
   `globals.css` 의 `@theme` 별칭이 참조하는 변수 이름을 뽑는다. 이 목록이 채워야 할 칸이다.

3. 디자인 표 읽기
   `dc.html` 에서 태그를 걷어낸 텍스트로 팔레트 (이름 · 쓰는 곳 · 값) 와 타이포 (글꼴 · 크기 · 굵기) 를 뽑는다. `README.md` 의 규칙 (그림자 사용 여부, 배경 겹침 금지 등) 도 함께 읽는다.

4. 대응표 만들고 확인받기
   토큰마다 디자인 표의 어느 항목을 넣을지 "쓰는 곳" 설명으로 정한다.

   | 토큰 | 디자인 표에서 찾는 것 |
   |---|---|
   | `--bg-primary` | 화면 전체 바탕 |
   | `--bg-surface`, `--bg-elevated` | 시트 · 모달 · 입력칸 안쪽 면 |
   | `--bg-secondary`, `--bg-tertiary` | 구획 바탕, 비활성 바탕. 없으면 배경에서 유도 |
   | `--accent-primary` | 주 버튼 칠 |
   | `--accent-secondary` | 주색의 진한 단계 |
   | `--accent-muted` | 주색 경계선 |
   | `--accent-glow`, `--accent-glow-strong` | 주색의 약한 단계 (칩 바탕, 선택된 항목 바탕) |
   | `--accent-warm` | 보조 신호색 (알림 점, 배지) |
   | `--text-primary` / `secondary` / `tertiary` | 본문 · 보조 · 비활성 글자 |
   | `--text-accent` | 라벨처럼 주색으로 쓰는 글자 |
   | `--border-subtle` / `default` / `hover` | 구분선 · 입력칸 테두리 · 포커스 테두리 |
   | `--shadow-*` | 그림자. 디자인이 그림자를 쓰지 않으면 `none` |
   | `--font-body` | 글꼴. globals.css 의 `--font-sans` 별칭이 이것을 가리킨다 |

   디자인 표에 대응 항목이 없는 토큰은 가까운 값에서 유도하고 주석에 "유도" 라고 적는다.
   디자인 표에는 있는데 들어갈 토큰이 없는 색은 확장 토큰 이름을 제안한다.
   대응표와 확장 토큰 목록을 사용자에게 보여주고 확인받는다.

5. 리소스 파일 쓰기
   `src/styles/themes/{name}.css` 를 만든다. 디자인에 다크 값이 없으면 라이트 전용으로 두고 파일 머리에 적는다.

6. 앱에 연결하기
   `globals.css` 에서 이전 테마 import 를 새 파일로 바꾸고, 확장 토큰의 `@theme` 별칭을 추가한다. 글꼴이 `next/font/google` 에 없으면 `layout.tsx` 에 `<link>` 를 넣는다. 1단계에서 "파일만 만들기" 를 골랐으면 이 단계를 건너뛴다.

7. 검증
   `npm run typecheck && npm run lint && npm run test:run`. 디자인 export 폴더가 레포 안에 있으면 eslint 무시 목록에 넣는다. 개발 서버로 홈을 띄워 배경 · 글자 색과 글꼴이 표의 값인지 본다.

### 종료 조건

- [ ] 입력 색 표의 모든 색이 리소스 파일에 들어갔다 (기본 토큰 또는 확장 토큰)
- [ ] Next.js 앱이 그 리소스 파일을 읽는다 (`globals.css` 가 import 하고, 홈 화면의 배경색이 표의 값이다). "파일만 만들기" 를 골랐으면 이 항목은 건너뛴다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- 팔레트 표가 여러 안이고 확정안 표시가 없다
- 같은 "쓰는 곳" 에 값이 둘 이상 있다
- 디자인에 다크 값이 없는데 사용자가 다크 모드를 원한다고 말한 적이 있다
