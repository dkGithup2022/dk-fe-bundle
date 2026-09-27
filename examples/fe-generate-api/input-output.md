# fe-generate-api 실행 예시

Peeple(지역 모임 · 이벤트 서비스) 을 이 체인으로 만든 실제 실행에서 발췌했다. 검수된 api.md 를 코드로 옮긴 결과. mock 함수마다 시나리오 주석을 달고, 테스트가 그 case 를 하나씩 확인한다.

## 입력

- 상태가 "검수 완료" 인 `docs/pages/{라우트}/api.md`
- `data.md` 의 API 데이터 절

## 사람이 정한 것

- 없음. 판단하지 않는 스킬이다

## 산출물

### mock 시나리오 주석 (참석 신청)

출처: `src/lib/attendance-data.ts`

```ts
/**
 * POST /api/events/:id/attend
 * case 1: 인자 { id: 자리가 남은 이벤트 (capacity null 이거나 attendeeCount < capacity) }
 *   동작: ATTENDING 레코드 추가, attendeeCount +1
 *   정상 응답: { status: "ATTENDING", attendeeCount: n+1, waitlistCount }
 * case 2: 인자 { id: 정원이 찬 이벤트 }
 *   동작: WAITLISTED 레코드 추가, waitlistCount +1
 *   정상 응답: { status: "WAITLISTED", attendeeCount, waitlistCount: n+1 }
 * case 3: 인자 { id: 이미 신청한 이벤트 }
 *   동작: 아무것도 바꾸지 않음
 *   정상 응답: 현재 상태와 수
 * case 4: 인자 { id: 없는 이벤트 }
 *   예외: 404 "이벤트를 찾을 수 없어요"
 */
```

### 호출 함수

출처: `src/lib/event-api.ts`

```ts
/** 페이지: 게시글 뷰어 (하단 바 [참석 신청] · [대기 신청]). 인증 필요. 지금은 mock 사용자로 동작 */
export const attendEvent = (eventId: number) => apiPost<AttendEventResponse>(`/api/events/${eventId}/attend`);
```
