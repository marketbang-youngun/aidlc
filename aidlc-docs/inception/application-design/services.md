# Services Design

## Service Layer Overview

```
Controller Layer
    │
    ▼
Service Layer (비즈니스 로직, 트랜잭션 관리)
    │
    ▼
Repository Layer (데이터 접근)
    │
    ▼
Database (MySQL)
```

---

## 1. GroupBuyAdminService

**책임**: 어드민 공구 CRUD + 상태 전환
**트랜잭션**: 쓰기 작업 @Transactional, 읽기 @Transactional(readOnly = true)

**주요 오케스트레이션:**
- `createGroupBuy`: 입력 검증 → GroupBuy 엔티티 생성 → 저장
- `cancelGroupBuy`: 상태 검증(RECRUITING만 가능) → MANUAL_CANCELLED 전환
- `prepareGroupBuy`: 상태 검증(CONFIRMED만 가능) → PREPARING 전환
- `completeGroupBuy`: 상태 검증(PREPARING만 가능) → COMPLETED 전환

---

## 2. GroupBuyBuyerService

**책임**: 유저용 공구 조회 (읽기 전용)
**트랜잭션**: @Transactional(readOnly = true)

**주요 오케스트레이션:**
- `getActiveGroupBuyList`: RECRUITING 상태 + 현재 시각이 start_at ~ end_at 사이인 공구 조회
- `getGroupBuyDetail`: 공구 기본 정보 + 최근 참여자 5명 포함
- `getGroupBuyStatus`: 폴링용 경량 응답 (current_quantity, participant_count, 최근 참여자)

---

## 3. ParticipationService

**책임**: 참여 신청/취소, 동시성 제어
**트랜잭션**: @Transactional (비관적 락 사용)

**주요 오케스트레이션:**
- `joinGroupBuy`:
  1. GroupBuy 조회 (비관적 락 FOR UPDATE)
  2. 상태 검증 (RECRUITING + 마감 전)
  3. 수량 검증 (max_quantity 초과 확인, min/max_per_user 검증)
  4. 중복 참여 확인 (동일 이메일)
  5. Participation 생성
  6. GroupBuy current_quantity, participant_count 갱신
- `cancelParticipation`:
  1. Participation 조회
  2. 상태 검증 (참여중 + 공구 RECRUITING)
  3. Participation 상태 → 취소, cancelled_at 설정
  4. GroupBuy current_quantity, participant_count 차감

---

## 4. ProductMockService

**책임**: URL 기반 Mock 상품 데이터 반환
**트랜잭션**: 없음

**주요 오케스트레이션:**
- `fetchProductByUrl`: URL에서 상품번호 추출 → 하드코딩된 Mock 데이터 맵에서 반환

---

## 5. GroupBuyScheduler

**책임**: 마감 공구 자동 확정/취소
**트랜잭션**: @Transactional (건별 처리)

**주요 오케스트레이션:**
- `processExpiredGroupBuys`:
  1. RECRUITING 상태 + end_at <= 현재시각인 공구 목록 조회
  2. 각 공구에 대해:
     - current_quantity >= target_quantity → CONFIRMED + 참여자 상태 확정
     - current_quantity < target_quantity → AUTO_CANCELLED

---

## Service 간 의존 관계

```
GroupBuyAdminService ──→ GroupBuyRepository
                     ──→ ParticipationRepository (참여자 조회)

GroupBuyBuyerService ──→ GroupBuyRepository
                     ──→ ParticipationRepository (최근 참여자)

ParticipationService ──→ ParticipationRepository
                     ──→ GroupBuyRepository (락 + 수량 갱신)

ProductMockService   ──→ (내부 Mock 데이터만)

GroupBuyScheduler    ──→ GroupBuyRepository
                     ──→ ParticipationRepository (확정 시 상태 일괄 변경)
```
