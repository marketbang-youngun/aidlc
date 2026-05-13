# Application Design — 공동구매 서비스

## 1. 시스템 개요

마켓뱅 공동구매 v2 MVP 데모 — 3-tier 모노레포 구조

| 유닛 | 기술 | 포트 | 담당 |
|------|------|------|------|
| Backend API | Spring Boot 3 + Java 17 + MySQL | :8888 | 백엔드 개발자 |
| Admin Front | Vue 3 + CoreUI + Vue CLI | :8081 | 프론트 개발자 |
| User Front | Vue 3 + Vite (모바일 우선) | :8080 | 프론트 개발자 |

---

## 2. Backend 컴포넌트 구조

```
com.threelabs/
├── config/security/         # JWT 인증 (간소화)
├── controller/
│   ├── admin/              # 어드민 API
│   ├── buyer/              # 유저 API (public)
│   └── common/             # 공통 API (상품 Mock)
├── service/
│   ├── GroupBuyAdminService
│   ├── GroupBuyBuyerService
│   ├── ParticipationService
│   └── ProductMockService
├── repository/
│   ├── GroupBuyRepository + Custom
│   ├── ParticipationRepository + Custom
│   └── AdminUserRepository
├── entity/
│   ├── GroupBuy
│   ├── Participation
│   └── AdminUser
├── dto/ (request/response)
├── enumration/
│   ├── GroupBuyStatus
│   └── ParticipationStatus
├── error/                   # 예외 처리
├── schedule/               # @Scheduled 마감 처리
└── config/                 # DB, QueryDSL, Swagger 설정
```

---

## 3. 핵심 설계 결정

| 항목 | 결정 | 이유 |
|------|------|------|
| 동시성 제어 | 비관적 락 (SELECT FOR UPDATE) | 참여 시 수량 정합성 보장 |
| 마감 처리 | @Scheduled 1분 간격 | 간단하고 데모에 충분 |
| 인증 (어드민) | JWT Cookie (간소화) | 기존 마켓뱅 패턴 유지 |
| 인증 (유저) | 이메일 파라미터 | 데모 간소화 |
| 상품 연동 | Mock 데이터 | 외부 API 의존 제거 |
| 실시간 현황 | HTTP 폴링 5초 | WebSocket 복잡도 회피 |
| 소프트 삭제 | YnCode.N | 기존 패턴 유지 |

---

## 4. 상태 머신

```
              +--- cancelGroupBuy() --→ [MANUAL_CANCELLED]
              |
[RECRUITING] -+--- Scheduler(목표달성) --→ [CONFIRMED] --→ [PREPARING] --→ [COMPLETED]
              |
              +--- Scheduler(목표미달) --→ [AUTO_CANCELLED]
```

---

## 5. API 설계 요약

- **Admin**: 9개 엔드포인트 (로그인 + CRUD + 상태전환 + 참여자 조회)
- **Buyer (Public)**: 6개 엔드포인트 (목록 + 상세 + 폴링 + 참여 + 취소 + 내역)
- **Common**: 1개 엔드포인트 (상품 Mock 조회)

상세: `component-methods.md` 참조

---

## 6. 프론트엔드 구조

### Admin Front
- 5개 화면: 로그인, 목록, 등록, 상세, 수정
- CoreUI 컴포넌트 사용 (CCard, CTable, CForm 등)
- Pinia 스토어: 공구 폼 상태 관리

### User Front
- 3개 화면 + 모달: 목록, 상세, 내 참여 내역
- 마켓뱅 디자인 시스템 적용 (버건디 #911054, 라임 #7fe816)
- Composable: usePolling (5초 간격 현황 갱신)
- 컴포넌트: ProgressBar, CountdownTimer, ParticipationModal, RecentParticipants

---

## 7. 관련 산출물

- [components.md](./components.md) — 컴포넌트 상세 목록
- [component-methods.md](./component-methods.md) — 메서드 시그니처 + API 엔드포인트
- [services.md](./services.md) — 서비스 레이어 설계 + 오케스트레이션
- [component-dependency.md](./component-dependency.md) — 의존 관계 + 데이터 플로우
