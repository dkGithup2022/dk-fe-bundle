# fe-apply-components 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 실행일 2026-09-21. 이후 페이지를 만들며 몇 개가 더해졌다.

## 입력

- 디자인 export 의 공용 컴포넌트 시트 (26개 + 공용으로 두지 않을 것 4건)
- 테마 파일과 `@theme` 별칭

## 사람이 정한 것

- 시트의 "변형으로 흡수" 3건(Tabs size, Badge dot, FormField suffix)은 새 파일이 아니라 옵션으로 둔다
- 도메인 타입이 필요한 조합(GroupRow, EventRow 등)은 `ui/` 가 아니라 `components/{domain}/` 에 두고 데이터 단계 뒤로 미룬다

## 산출물

### 만들어진 ui 컴포넌트

출처: `src/components/ui/`

```text
AppBar AvatarStack Badge BottomSheet Button Calendar Checkbox Chip DateBlock DayPicker Divider EmptyState FAB FormField Label ListRow RadioGroup SectionHeader SelectField StepHeader StepProgressHeader Tabs Textarea Thumb TimeRow TitleMetaRow Toggle ToneLabel
```

### 기준 모양 (Button)

출처: `src/components/ui/Button.tsx`

```tsx
import { cn } from "@/lib/utils";
import type { ButtonProps, ButtonSize, ButtonVariant } from "./Button.types";

const variantClass: Record<ButtonVariant, string> = {
  primary: "bg-accent text-text-on-accent",
  strong: "bg-accent-secondary text-surface",
  outline: "border border-accent-muted text-foreground",
  text: "text-text-secondary",
};

const sizeClass: Record<ButtonSize, string> = {
  sm: "h-8 px-3 text-desc",
  md: "h-10 px-4 text-body",
  lg: "h-12 px-5 text-body",
};

export function Button({ variant = "primary", size = "md", fullWidth, selected, className, ...rest }: ButtonProps) {
  return (
    <button
      type="button"
      className={cn(
 "inline-flex items-center justify-center rounded-full font-bold whitespace-nowrap",
 "disabled:bg-tertiary disabled:text-text-tertiary disabled:border-transparent",
        variantClass[variant],
        sizeClass[size],
        variant === "outline" && selected && "bg-accent-glow",
        fullWidth && "w-full",
        className,
      )}
      {...rest}
    />
  );
}
```
