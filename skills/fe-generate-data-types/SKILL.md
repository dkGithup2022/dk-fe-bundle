---
name: fe-generate-data-types
description: 사용자가 검수한 docs/data-spec.md 를 읽어 src/types/data/domain/ 과 src/types/data/feature/ 에 타입 파일을 만든다. 스펙에 없는 것은 만들지 않는다.
argument-hint: "[데이터 스펙 파일 (기본: 가장 높은 판의 docs/data-spec*.md)]"
---

# 데이터 타입 생성

> 산출물(`docs/`, `src/` 아래 파일)은 플러그인 폴더가 아니라 **현재 작업 중인 프로젝트** 아래에 만든다. 스킬 본문의 경로는 모두 그 프로젝트 기준이다.

## 에이전틱 동작

이 동작은 에이전틱한 동작입니다. 아래 순서에 맞게 실행하되, 수행 계획과 종료 조건을 고려하여 여러 번 수행해 결과를 도출해도 됩니다.

### 목적

검수가 끝난 스펙 문서를 그대로 타입 파일로 옮기는 것이 목적입니다. 판단은 하지 않습니다. 스펙에 없는 필드를 더하거나 이름을 바꾸지 않습니다.

### 입력 & 출력

입력:
- `docs/data-spec.md`. 여러 판이 있으면 가장 높은 번호의 `docs/data-spec-v{n}.md`. 인자로 다른 파일을 받을 수도 있다.
- `src/types/data/` 의 현재 파일. 이미 있는 것은 스펙과 다른 부분만 고친다.

출력:
- `src/types/data/domain/{name}.ts`: 도메인마다 하나. 값 타입도 여기.
- `src/types/data/feature/{feature}.ts`: 화면 분기용 enum 과 기능 저장 데이터. 기능 이름마다 하나.
- 파일 머리에 스펙의 한 줄 설명과 근거 화면, 필드마다 근거 문구를 주석으로.

```ts
// src/types/data/domain/{name}.ts
import type { {ValueType} } from "./{value-type}";

/**
 * {항목 이름} (도메인) — {1줄 설명}
 * 근거 화면: {파일명}, {파일명}
 * 보류: {스펙의 보류 항목 중 이 도메인에 관한 것}
 */
export interface {Name} {
  id: number;
  {field}: {type};          // {파일명} "{표시 문구}"
  {field}: {type} | null;   // {파일명} "{표시 문구}". null 이면 {뜻}
  {field}: {EnumName};      // {파일명} "{표시 문구}"
}

/** {enum 설명} (enum) — {파일명} */
export type {EnumName} = "{VALUE_A}" | "{VALUE_B}";
export const {ENUM_NAME}_LABEL: Record<{EnumName}, string> = {
  {VALUE_A}: "{화면 문구}",
  {VALUE_B}: "{화면 문구}",
};
```

```ts
// src/types/data/feature/{feature}.ts
/**
 * {기능 이름} 흐름 (기능 데이터) — {화면 분기에만 쓴다. 저장되는 필드가 아니다 | 저장 단위: 사용자별}
 * 근거 화면: {파일명}, {파일명}
 */
export type {EnumName} = "{VALUE_A}" | "{VALUE_B}";
export const {ENUM_NAME}_LABEL: Record<{EnumName}, string> = { ... };

export interface {StoredName} {
  {field}: {type}; // {파일명} "{표시 문구}"
}
```

### 배치 규칙

- 도메인 → `domain/{kebab-name}.ts`.
- 값 타입 → `domain/{kebab-name}.ts`. 쓰는 도메인이 import 한다.
- enum 의 "쓰는 곳" 이 한 도메인의 필드 → 그 도메인 파일 안.
- enum 의 "쓰는 곳" 이 두 도메인 이상의 필드 → `domain/{kebab-name}.ts` 로 따로.
- enum 의 "쓰는 곳" 이 기능의 화면 분기 → `feature/{기능 이름}.ts`. 같은 기능의 enum 은 한 파일에.
- 기능 저장 데이터 → `feature/{기능 이름}.ts`.
- 기능 하나에 파일이 둘 이상 필요해지면 `feature/{기능 이름}/` 폴더로 올린다.
- enum 은 유니온 타입 + `{ENUM_NAME}_LABEL` 상수 한 쌍. 값이 미정이면 `string` 으로 두고 주석에 적는다.
- 스펙의 "결정 필요" 가 비어 있지 않으면 시작하지 않는다.

### 권장 동작 순서

1. 스펙 읽기
   "결정 필요" 가 남아 있으면 사용자에게 알리고 멈춘다. 항목 · 필드 · enum · 쓰는 곳 · 보류를 뽑는다.

2. 배치 정하기
   배치 규칙대로 항목마다 파일 경로를 정한다. `src/types/data/` 에 이미 있는 파일과 대조해 새로 만듦 / 고침 / 그대로 를 표시한다.

3. 파일 쓰기
   표의 근거 열을 필드 주석으로 옮긴다. 스펙에 없는 필드는 넣지 않는다.

4. 검증
   `npm run typecheck && npm run lint && npm run test:run`.

### 종료 조건

- [ ] 스펙의 모든 항목이 파일로 있고, 파일의 모든 필드가 스펙에 있다
- [ ] enum 마다 유니온 타입과 라벨 상수가 있다
- [ ] 검증 명령을 통과했다

아래 상황이면 AskUser 로 상황을 브리핑하고 정보를 구하세요.
- 스펙의 "결정 필요" 가 비어 있지 않다
- enum 에 "쓰는 곳" 이 없어 파일 위치를 정할 수 없다
- 이미 있는 파일의 필드가 스펙과 달라 어느 쪽이 맞는지 알 수 없다
