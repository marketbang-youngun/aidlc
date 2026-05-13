# Component Methods

## Backend API Methods

---

### GroupBuyAdminService

| 메서드 | 입력 | 출력 | 설명 |
|--------|------|------|------|
| `createGroupBuy(dto)` | `GroupBuyCreateRequestDto` | `Long (id)` | 공구 등록 |
| `getGroupBuyList(dto)` | `GroupBuyListRequestDto` (status, pageable) | `Page<GroupBuyListResponseDto>` | 목록 조회 |
| `getGroupBuyDetail(id)` | `Long` | `GroupBuyDetailResponseDto` | 상세 조회 |
| `updateGroupBuy(id, dto)` | `Long`, `GroupBuyUpdateRequestDto` | `void` | 수정 |
| `cancelGroupBuy(id)` | `Long` | `void` | 수동 취소 |
| `prepareGroupBuy(id)` | `Long` | `void` | 배송준비 전환 |
| `completeGroupBuy(id)` | `Long` | `void` | 배송완료 처리 |
| `getParticipants(id)` | `Long` | `List<ParticipantResponseDto>` | 참여자 목록 |

---

### GroupBuyBuyerService

| 메서드 | 입력 | 출력 | 설명 |
|--------|------|------|------|
| `getActiveGroupBuyList()` | 없음 | `List<GroupBuyCardResponseDto>` | 모집중 목록 |
| `getGroupBuyDetail(id)` | `Long` | `GroupBuyBuyerDetailResponseDto` | 유저용 상세 |
| `getGroupBuyStatus(id)` | `Long` | `GroupBuyStatusResponseDto` | 폴링용 현황 |

---

### ParticipationService

| 메서드 | 입력 | 출력 | 설명 |
|--------|------|------|------|
| `joinGroupBuy(groupbuyId, dto)` | `Long`, `ParticipationJoinRequestDto` | `ParticipationResponseDto` | 참여 신청 |
| `cancelParticipation(groupbuyId, dto)` | `Long`, `ParticipationCancelRequestDto` | `void` | 참여 취소 |
| `getMyParticipations(email)` | `String` | `List<MyParticipationResponseDto>` | 내 참여 내역 |

---

### ProductMockService

| 메서드 | 입력 | 출력 | 설명 |
|--------|------|------|------|
| `fetchProductByUrl(url)` | `String` | `ProductFetchResponseDto` | URL로 Mock 상품 조회 |

---

### GroupBuyScheduler

| 메서드 | 입력 | 출력 | 설명 |
|--------|------|------|------|
| `processExpiredGroupBuys()` | 없음 | `void` | 1분 간격, 마감 공구 확정/취소 처리 |

---

## REST API Endpoints

### Admin API (`/api/admin/v1/`)

| Method | Path | Service Method | 설명 |
|--------|------|---------------|------|
| POST | `/user/login` | (Security Filter) | 로그인 |
| POST | `/groupbuy` | `createGroupBuy` | 공구 등록 |
| GET | `/groupbuy/list` | `getGroupBuyList` | 목록 |
| GET | `/groupbuy/{id}` | `getGroupBuyDetail` | 상세 |
| PUT | `/groupbuy/{id}` | `updateGroupBuy` | 수정 |
| POST | `/groupbuy/{id}/cancel` | `cancelGroupBuy` | 수동 취소 |
| POST | `/groupbuy/{id}/prepare` | `prepareGroupBuy` | 배송준비 |
| POST | `/groupbuy/{id}/complete` | `completeGroupBuy` | 배송완료 |
| GET | `/groupbuy/{id}/participants` | `getParticipants` | 참여자 |

### Buyer API (`/api/buyer/v1/public/`)

| Method | Path | Service Method | 설명 |
|--------|------|---------------|------|
| GET | `/groupbuy/list` | `getActiveGroupBuyList` | 진행중 목록 |
| GET | `/groupbuy/{id}` | `getGroupBuyDetail` | 상세 |
| GET | `/groupbuy/{id}/status` | `getGroupBuyStatus` | 폴링용 현황 |
| POST | `/groupbuy/{id}/join` | `joinGroupBuy` | 참여 |
| POST | `/groupbuy/{id}/cancel` | `cancelParticipation` | 참여 취소 |
| GET | `/groupbuy/my` | `getMyParticipations` | 내 내역 (email param) |

### Common API (`/api/common/v1/`)

| Method | Path | Service Method | 설명 |
|--------|------|---------------|------|
| GET | `/product/fetch` | `fetchProductByUrl` | 상품 Mock 조회 (url param) |
