# Component Dependency

## System Architecture

```
+------------------+     +-------------------+
|  Admin Front     |     |   User Front      |
|  (Vue3+CoreUI)   |     |   (Vue3+Vite)     |
|  :8081           |     |   :8080           |
+--------+---------+     +---------+---------+
         |                          |
         |  HTTP (REST API)         |  HTTP (REST API)
         |                          |
+--------+--------------------------+---------+
|              Backend API                     |
|           (Spring Boot 3, :8888)            |
|                                             |
|  +------------------+  +----------------+  |
|  | Admin API        |  | Buyer API      |  |
|  | /api/admin/v1/** |  | /api/buyer/v1/ |  |
|  |                  |  | public/**      |  |
|  +--------+---------+  +-------+--------+  |
|           |                     |           |
|  +--------+---------------------+--------+  |
|  |          Service Layer               |   |
|  | GroupBuyAdminService                 |   |
|  | GroupBuyBuyerService                 |   |
|  | ParticipationService                 |   |
|  | ProductMockService                   |   |
|  | GroupBuyScheduler                    |   |
|  +--------+-----------------------------+   |
|           |                                 |
|  +--------+-----------------------------+   |
|  |          Repository Layer            |   |
|  | GroupBuyRepository                   |   |
|  | ParticipationRepository              |   |
|  | AdminUserRepository                  |   |
|  +--------+-----------------------------+   |
|           |                                 |
+-----------+---------------------------------+
            |
   +--------+--------+
   |    MySQL DB      |
   |    (단일 Master) |
   +-----------------+
```

---

## Dependency Matrix

| Component | Depends On |
|-----------|-----------|
| AdminGroupBuyController | GroupBuyAdminService, LoginUserProviderUtil |
| BuyerGroupBuyController | GroupBuyBuyerService |
| BuyerParticipationController | ParticipationService |
| CommonProductController | ProductMockService |
| GroupBuyAdminService | GroupBuyRepository, ParticipationRepository |
| GroupBuyBuyerService | GroupBuyRepository, ParticipationRepository |
| ParticipationService | ParticipationRepository, GroupBuyRepository |
| ProductMockService | (없음 — 내부 Mock) |
| GroupBuyScheduler | GroupBuyRepository, ParticipationRepository |
| SecurityConfig | JwtTokenProvider, CustomUserDetailsService, AdminUserRepository |

---

## Communication Patterns

### Frontend → Backend
- **프로토콜**: HTTP REST (JSON)
- **인증 (Admin)**: JWT Cookie (`marketbang_admin_access_token`)
- **인증 (User)**: 없음 (public API), 이메일 파라미터로 식별
- **폴링**: User Front → `GET /api/buyer/v1/public/groupbuy/{id}/status` (5초 간격)

### Frontend CORS 설정
```
Admin Front (:8081) → Backend (:8888)
User Front  (:8080) → Backend (:8888)
```

---

## Data Flow

### 공구 등록 플로우
```
Admin Front → POST /api/admin/v1/groupbuy
    → AdminGroupBuyController.createGroupBuy()
    → GroupBuyAdminService.createGroupBuy()
    → GroupBuyRepository.save()
    → MySQL
```

### 공구 참여 플로우
```
User Front → POST /api/buyer/v1/public/groupbuy/{id}/join
    → BuyerParticipationController.joinGroupBuy()
    → ParticipationService.joinGroupBuy()
    → GroupBuyRepository.findByIdForUpdate() [비관적 락]
    → 검증 (수량, 상태, 중복)
    → ParticipationRepository.save()
    → GroupBuy.currentQuantity++
    → GroupBuyRepository.save()
    → MySQL
```

### 마감 자동 처리 플로우
```
@Scheduled(fixedRate = 60000)
    → GroupBuyScheduler.processExpiredGroupBuys()
    → GroupBuyRepository.findByStatusAndEndAtBefore(RECRUITING, now)
    → 각 공구: 수량 판단 → CONFIRMED or AUTO_CANCELLED
    → ParticipationRepository.updateStatusByGroupBuyId() [확정 시]
    → MySQL
```

### 폴링 플로우
```
User Front → setInterval(5000)
    → GET /api/buyer/v1/public/groupbuy/{id}/status
    → GroupBuyBuyerService.getGroupBuyStatus()
    → 경량 응답 (currentQuantity, participantCount, recentParticipants)
    → UI 갱신
```
