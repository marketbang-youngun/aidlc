# Unit 1: Backend — Functional Design

## 1. 상태 머신 (GroupBuy Status)

```
RECRUITING → CONFIRMED → PREPARING → COMPLETED
           → AUTO_CANCELLED
           → MANUAL_CANCELLED
```

### 전환 규칙
| From | To | 트리거 | 조건 |
|------|----|--------|------|
| RECRUITING | CONFIRMED | Scheduler | end_at <= now AND current_quantity >= target_quantity |
| RECRUITING | AUTO_CANCELLED | Scheduler | end_at <= now AND current_quantity < target_quantity |
| RECRUITING | MANUAL_CANCELLED | Admin API | 운영자 수동 취소 |
| CONFIRMED | PREPARING | Admin API | 운영자 배송준비 전환 |
| PREPARING | COMPLETED | Admin API | 운영자 배송완료 |

---

## 2. 동시성 제어 — 참여 신청

### 비관적 락 전략
```sql
SELECT * FROM group_buy WHERE id = ? FOR UPDATE
```

### 참여 처리 순서
1. GroupBuy 조회 (FOR UPDATE — 행 락)
2. 상태 검증: status == RECRUITING AND now < end_at
3. 수량 검증: current_quantity + request_quantity <= max_quantity (or unlimited)
4. 개인 수량 검증: min_per_user <= quantity <= max_per_user
5. 중복 검증: 동일 email + 동일 groupbuy_id + status=ACTIVE 없음
6. Participation 저장
7. GroupBuy.current_quantity += quantity, participant_count++
8. 커밋 (락 해제)

---

## 3. 마감 자동 처리

### 스케줄러 로직
```
매 1분 실행:
  expired = GroupBuy WHERE status='RECRUITING' AND end_at <= NOW()
  FOR EACH gb IN expired:
    IF gb.current_quantity >= gb.target_quantity:
      gb.status = CONFIRMED
      UPDATE Participation SET status='CONFIRMED' WHERE groupbuy_id=gb.id AND status='ACTIVE'
    ELSE:
      gb.status = AUTO_CANCELLED
```

---

## 4. 비즈니스 규칙

### 공구 등록 검증
- 종료일시 > 시작일시
- target_quantity > 0
- max_quantity == null OR max_quantity >= target_quantity
- groupbuy_price > 0
- min_per_user >= 1
- max_per_user == null OR max_per_user >= min_per_user

### 참여 취소 검증
- Participation.status == ACTIVE
- GroupBuy.status == RECRUITING
- 취소 후: current_quantity -= quantity, participant_count--

### 공구 수정 검증
- GroupBuy.status == RECRUITING만 수정 가능

---

## 5. Mock 상품 데이터

URL 패턴: `https://marketbang.kr/detail/{id}`
상품번호 추출 후 하드코딩된 5개 Mock 상품 중 하나 반환.
존재하지 않는 ID → 에러 응답.
