# Application Components

## Backend Components (Spring Boot 3)

### 1. GroupBuy Component
**패키지**: `com.threelabs.groupbuy`
**책임**: 공동구매 CRUD, 상태 관리, 라이프사이클 처리

| 하위 | 역할 |
|------|------|
| `entity/GroupBuy.java` | 공구 엔티티 |
| `repository/GroupBuyRepository.java` | JPA Repository |
| `repository/CustomGroupBuyRepository.java` | QueryDSL 커스텀 |
| `repository/CustomGroupBuyRepositoryImpl.java` | QueryDSL 구현 |
| `dto/admin/request/` | 어드민 요청 DTO |
| `dto/admin/response/` | 어드민 응답 DTO |
| `dto/buyer/response/` | 유저 응답 DTO |
| `service/GroupBuyAdminService.java` | 어드민 비즈니스 로직 |
| `service/GroupBuyBuyerService.java` | 유저 조회 로직 |
| `controller/admin/AdminGroupBuyController.java` | 어드민 API |
| `controller/buyer/BuyerGroupBuyController.java` | 유저 API |

---

### 2. Participation Component
**패키지**: `com.threelabs.participation`
**책임**: 공구 참여/취소, 참여 현황 조회

| 하위 | 역할 |
|------|------|
| `entity/Participation.java` | 참여 엔티티 |
| `repository/ParticipationRepository.java` | JPA Repository |
| `repository/CustomParticipationRepository.java` | QueryDSL 커스텀 |
| `repository/CustomParticipationRepositoryImpl.java` | QueryDSL 구현 |
| `dto/request/` | 참여 요청 DTO |
| `dto/response/` | 참여 응답 DTO |
| `service/ParticipationService.java` | 참여 비즈니스 로직 |
| `controller/buyer/BuyerParticipationController.java` | 유저 참여 API |

---

### 3. Product Mock Component
**패키지**: `com.threelabs.product`
**책임**: 마켓뱅 상품 URL 기반 Mock 데이터 제공

| 하위 | 역할 |
|------|------|
| `dto/response/ProductFetchResponseDto.java` | 상품 정보 응답 DTO |
| `service/ProductMockService.java` | Mock 상품 데이터 생성 |
| `controller/common/CommonProductController.java` | 상품 조회 API |

---

### 4. Admin Auth Component
**패키지**: `com.threelabs.config.security`
**책임**: 어드민 로그인 (간소화 JWT Cookie)

| 하위 | 역할 |
|------|------|
| `entity/AdminUser.java` (in admin package) | 어드민 사용자 엔티티 |
| `repository/AdminUserRepository.java` | JPA Repository |
| `SecurityConfig.java` | Spring Security 설정 |
| `filter/JwtAuthenticationFilter.java` | 로그인 필터 |
| `filter/JwtAuthorizationFilter.java` | 인가 필터 |
| `provider/JwtTokenProvider.java` | 토큰 생성/검증 |
| `user_details/CustomUserDetails.java` | UserDetails 구현 |
| `user_details/CustomUserDetailsService.java` | UserDetailsService |

---

### 5. Scheduler Component
**패키지**: `com.threelabs.schedule`
**책임**: 공구 마감 자동 처리 (1분 간격)

| 하위 | 역할 |
|------|------|
| `GroupBuyScheduler.java` | @Scheduled 마감 처리 |

---

### 6. Common/Config Component
**패키지**: `com.threelabs.config`, `com.threelabs.dto`, `com.threelabs.error`, `com.threelabs.enumration`
**책임**: 공통 설정, 응답 포맷, 예외 처리, Enum

| 하위 | 역할 |
|------|------|
| `dto/ResponseDto.java` | 공통 응답 래퍼 |
| `error/` | 예외 처리 (CustomErrorCodeException, Handler) |
| `enumration/GroupBuyStatus.java` | 공구 상태 Enum |
| `enumration/ParticipationStatus.java` | 참여 상태 Enum |
| `enumration/code/YnCode.java` | Y/N Enum |
| `config/querydsl/QueryDslConfig.java` | JPAQueryFactory 빈 |

---

## Frontend Components

### Admin Front (Vue 3 + CoreUI)

| 컴포넌트 | 역할 |
|----------|------|
| `views/Login.vue` | 로그인 페이지 |
| `views/groupbuy/GroupBuyList.vue` | 공구 목록 (상태 탭) |
| `views/groupbuy/GroupBuyCreate.vue` | 공구 등록 (URL 연동 + 폼) |
| `views/groupbuy/GroupBuyDetail.vue` | 공구 상세 (참여 현황) |
| `views/groupbuy/GroupBuyEdit.vue` | 공구 수정 |
| `api/groupbuy.js` | API 호출 모듈 |
| `api/auth.js` | 인증 API |
| `pinia/groupbuy.js` | 공구 상태 스토어 |

### User Front (Vue 3 + Vite)

| 컴포넌트 | 역할 |
|----------|------|
| `views/GroupBuyList.vue` | 공구 목록 (카드형) |
| `views/GroupBuyDetail.vue` | 공구 상세 (프로그레스바, 카운트다운) |
| `views/MyParticipations.vue` | 내 참여 내역 |
| `components/ProgressBar.vue` | 달성률 프로그레스바 |
| `components/CountdownTimer.vue` | 카운트다운 타이머 |
| `components/ParticipationModal.vue` | 참여 모달 (수량, 이메일) |
| `components/RecentParticipants.vue` | 최근 참여자 알림 |
| `api/groupbuy.js` | API 호출 모듈 |
| `composables/usePolling.js` | 5초 폴링 composable |
