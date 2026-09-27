---
name: fe-apply-components
description: 디자인 핸드오프의 공용 컴포넌트 시트를 읽어 src/components/ui/ 에 컴포넌트 파일을 새로 만든다. 색은 테마 토큰만 쓴다.
argument-hint: "[디자인 export 폴더]"
---

# 공용 컴포넌트 만들기

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

디자인 시트의 공용 컴포넌트를 코드로 옮겨, 페이지가 색 · 크기 · 간격을 직접 쓰지 않고 컴포넌트를 조립만 하게 하는 것이 목적입니다.

### 입력 & 출력

입력:
- 사용자가 지정한 디자인 export 폴더. 컴포넌트 시트가 있는 `*.dc.html` 과 `README.md`. `support.js` 는 읽지 않는다.
- `src/components/ui/` 와 `src/components/{domain}/` 에 이미 있는 파일.
- `src/app/globals.css` 의 `@theme` 별칭과 그것이 가리키는 테마 파일.
- 대화에서 사용자가 정한 사항.

출력:
- `src/components/ui/{Name}.tsx` 와 `src/components/ui/{Name}.types.ts`: 컴포넌트마다 한 쌍.
- 동작이 하나로 정해진 컴포넌트는 `src/components/{domain}/{Name}.tsx` 와 `{Name}.types.ts`.
- 상태나 이벤트가 있는 컴포넌트는 `{Name}.test.tsx` 를 같이 둔다.
- `src/app/dev/ui/page.tsx` + `UiGalleryClient.tsx` + `loading.tsx`: 확인 페이지. 모든 컴포넌트를 변형별로 늘어놓는다. production 에서는 `notFound()`.

```ts
// src/components/ui/Button.types.ts
import type { ButtonHTMLAttributes } from "react";

export type ButtonVariant = "primary" | "strong" | "outline" | "text";
export type ButtonSize = "sm" | "md" | "lg";

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
  fullWidth?: boolean;
}
```

```tsx
// src/components/ui/Button.tsx
import { cn } from "@/lib/utils";
import type { ButtonProps, ButtonSize, ButtonVariant } from "./Button.types";

const variantClass: Record<ButtonVariant, string> = {
  primary: "bg-accent text-text-on-accent",
  strong: "bg-accent-secondary text-background",
  outline: "border border-accent-muted text-foreground",
  text: "text-text-secondary",
};

const sizeClass: Record<ButtonSize, string> = {
  sm: "h-8 px-3 text-[12.5px]",
  md: "h-10 px-4 text-[13px]",
  lg: "h-12 px-5 text-[13px]",
};

export function Button({ variant = "primary", size = "md", fullWidth, className, ...rest }: ButtonProps) {
  return (
    <button
      type="button"
      className={cn(
        "inline-flex items-center justify-center rounded-full font-bold disabled:opacity-50",
        variantClass[variant],
        sizeClass[size],
        fullWidth && "w-full",
        className,
      )}
      {...rest}
    />
  );
}
```

```tsx
// src/components/ui/Chip.test.tsx
import { it, expect, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Chip } from "./Chip";

it("클릭하면 onToggle 이 불린다", async () => {
  const onToggle = vi.fn();
  render(<Chip label="소셜" selected={false} onToggle={onToggle} />);
  await userEvent.click(screen.getByRole("button", { name: "소셜" }));
  expect(onToggle).toHaveBeenCalledOnce();
});
```

### 작성 규칙

인터랙션을 어디에 둘지:
- 동작이 쓰는 곳마다 다르면 콜백 prop 으로 받는다. 안에 로직을 두지 않는다. (Button, Chip, Toggle, ListRow)
- 동작이 하나로 정해져 있으면 컴포넌트 안에 로직을 둔다. 이런 것은 `components/{domain}/` 에 둔다. (OAuth 로그인 버튼, 팔로우 버튼)
- 어느 쪽인지 애매하면 사용자에게 묻는다.

최소로 만든다:
- 시트에 없는 prop, 변형, 상태는 만들지 않는다.
- 감싸기만 하는 div, index.ts 배럴, 기본값만 바꾸는 래퍼를 두지 않는다.
- UI 라이브러리(shadcn, radix 등)를 넣지 않는다. Tailwind 클래스와 토큰만 쓴다.
- 변형은 `variant` / `size` 유니온과 `Record` 맵으로 표현한다.
- 기준 모양은 위 Button 예시다.

크기:
- 전체 컨테이너와 각 하위 컴포넌트와 구성 요소는 고정 크기보다는 flex 를 사용하는 것이 권장됩니다.
- 크기를 직접 선언해야 한다면 숫자를 넣기 보다는 tailwind 변수에 제안된 크기 숫자가 있는지 확인하세요.

그 밖에:
- 파일 하나에 컴포넌트 하나. named export. props 타입은 같은 폴더의 `{Name}.types.ts`.
- `ui/` 는 `@/types` 를 import 하지 않는다. props 는 문자열 · 숫자 · 불리언 · 콜백 · ReactNode 만.
- 색은 `@theme` 별칭 (`bg-accent`, `text-text-secondary`) 또는 `var(--token)` 만 쓴다. 크기 · 굵기 · 둥글기는 시트 값을 Tailwind 클래스로 쓴다.
- `"use client"` 는 상태나 이벤트 핸들러가 있을 때만 붙인다.
- 접근성: 눌리는 것은 `button`, 토글은 `role="switch"` + `aria-checked`, 선택 칩은 `aria-pressed`, 입력칸은 `label` 과 연결한다.
- 테스트는 상태나 이벤트가 있는 컴포넌트에만 만든다.

### 권장 동작 순서

1. 기존 컴포넌트 확인
   `src/components/ui/` 와 `src/components/{domain}/` 의 export 이름과 props 를 뽑는다. 이미 있는 것은 새로 만들지 않고 시트와 다른 부분만 고친다.

2. 시트 읽기
   컴포넌트마다 이름, 변형 (variant · size · 상태), 쓰이는 곳, 예시 문구를 뽑는다. 시트가 "기존 컴포넌트에 옵션으로 넣는다" 고 표시한 항목은 새 파일이 아니라 그 컴포넌트의 prop 으로 적는다. "공용으로 두지 않을 것" 은 제외 목록에 적는다.

3. 작업 목록 정리
   시트 항목마다 새로 만듦 / 기존 수정 / 도메인 폴더 / 제외 중 하나를 정하고 props 초안 (이름 · 타입) 을 적는다. 사용자에게 보여주고 확인받는다.

4. 만들기
   작성 규칙을 따른다. 의존이 없는 것부터 만든다. 예: Divider, Label, Badge, Button, Chip, Thumb → FormField, ListRow → 이들을 조합한 것.

5. 시트와 대조
   확인 페이지에 시트 순서대로, 시트의 예시 문구로 모든 변형을 늘어놓는다. 개발 서버로 띄워 시트와 나란히 보고 색 · 크기 · 상태가 다른 곳을 고친다. 상태가 있는 컴포넌트는 테스트에서도 시트의 예시 문구로 렌더한다.

6. 검증
   `npm run typecheck && npm run lint && npm run test:run`.

### 종료 조건

- [ ] 시트의 모든 항목이 파일로 존재하거나 제외 결정이 기록되었다
- [ ] 검증 명령을 통과했다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- 컴포넌트의 동작이 쓰는 곳마다 다른지, 하나로 정해진 것인지 애매하다
- 시트의 변형이 그림으로만 있고 크기 · 색 값이 없다
- 같은 이름의 컴포넌트가 이미 있는데 props 가 시트와 다르다
- 시트가 요구하는 색이 토큰에 없다 (fe-apply-theme 로 돌아가 확장 토큰을 추가할지 정한다)
