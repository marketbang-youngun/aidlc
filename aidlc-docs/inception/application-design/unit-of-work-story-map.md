# Unit of Work — Story Map

## Story → Unit Mapping

### Unit 1: Backend API

| Story ID | 제목 | Priority | Backend 역할 |
|----------|------|----------|-------------|
| AS-01 | 어드민 로그인 | Must | JWT 인증 처리 |
| AS-02 | 공구 등록 — 상품 연동 | Must | Mock 상품 API |
| AS-03 | 공구 등록 — 공구 설정 | Must | GroupBuy CRUD |
| AS-04 | 공구 목록 조회 | Must | 목록 API (상태 필터) |
| AS-05 | 공구 상세 (참여 현황) | Must | 상세 + 참여자 API |
| AS-06 | 공구 수정 | Should | 수정 API |
| AS-07 | 수동 취소/완료 | Must | 상태 전환 API |
| US-01 | 진행중 목록 | Must | 공개 목록 API |
| US-02 | 공구 상세 | Must | 공개 상세 API |
| US-03 | 참여 신청 | Must | 참여 API + 동시성 제어 |
| US-04 | 참여 취소 | Must | 취소 API |
| US-05 | 내 참여 내역 | Should | 이메일 기반 조회 API |
| US-06 | 확정/취소 결과 | Must | 상태 포함 응답 |
| SS-01 | 마감 자동 처리 | Must | @Scheduled 스케줄러 |
| SS-02 | 실시간 폴링 | Should | 폴링용 경량 API |
| SS-03 | 동시성 제어 | Must | 비관적 락 |

**모든 스토리(16개)가 Backend에 관여**

---

### Unit 2: Frontend

#### Admin Front 담당 스토리

| Story ID | 제목 | Priority | Frontend 역할 |
|----------|------|----------|--------------|
| AS-01 | 어드민 로그인 | Must | 로그인 폼 + 쿠키 처리 |
| AS-02 | 공구 등록 — 상품 연동 | Must | URL 입력 + 자동채움 UI |
| AS-03 | 공구 등록 — 공구 설정 | Must | 등록 폼 + 검증 |
| AS-04 | 공구 목록 조회 | Must | 상태 탭 + 테이블 |
| AS-05 | 공구 상세 (참여 현황) | Must | 상세 + 참여자 리스트 |
| AS-06 | 공구 수정 | Should | 수정 폼 |
| AS-07 | 수동 취소/완료 | Must | 상태 전환 버튼 |

#### User Front 담당 스토리

| Story ID | 제목 | Priority | Frontend 역할 |
|----------|------|----------|--------------|
| US-01 | 진행중 목록 | Must | 카드 리스트 + 달성률 |
| US-02 | 공구 상세 | Must | 프로그레스바 + 카운트다운 |
| US-03 | 참여 신청 | Must | 수량 선택 + 이메일 입력 모달 |
| US-04 | 참여 취소 | Must | 취소 확인 다이얼로그 |
| US-05 | 내 참여 내역 | Should | 이메일 입력 + 목록 |
| US-06 | 확정/취소 결과 | Must | 상태 안내 메시지 |
| SS-02 | 실시간 폴링 | Should | usePolling composable |

---

## Coverage Summary

| Unit | Must Stories | Should Stories | Total |
|------|-------------|---------------|-------|
| Unit 1: Backend | 13 | 3 | 16 (전체) |
| Unit 2: Admin Front | 6 | 1 | 7 |
| Unit 2: User Front | 5 | 2 | 7 |

**모든 16개 스토리가 유닛에 할당됨. 누락 없음.**

---

## Construction Phase 실행 순서

CONSTRUCTION에서는 Per-Unit Loop를 유닛별로 실행합니다:

```
Unit 1: Backend API
  → Functional Design (상태 머신, 동시성, 비즈니스 규칙)
  → Code Generation (Planning → Generation)

Unit 2: Frontend
  → Functional Design (화면 흐름, 컴포넌트 상호작용)
  → Code Generation (Planning → Generation)

(모든 유닛 완료 후)
  → Build and Test
```
