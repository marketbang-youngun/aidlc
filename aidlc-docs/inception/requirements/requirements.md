# 공동구매 서비스 요구사항 문서

## Intent Analysis

| 항목 | 내용 |
|------|------|
| **사용자 요청** | 마켓뱅 공동구매 v2 MVP 데모 서비스 개발 |
| **요청 유형** | New Project (신규 프로젝트) |
| **범위** | Multiple Components (백엔드 API + 어드민 프론트 + 유저 프론트) |
| **복잡도** | Moderate |
| **프로젝트 성격** | 워크샵 데모 → 추후 운영 서비스 이식 가능 수준 |

---

## 1. 기술 스택 결정사항

### 1.1 백엔드
| 항목 | 기술 |
|------|------|
| 언어 | Java 17 |
| 프레임워크 | Spring Boot 3.5.x |
| ORM | Spring Data JPA + QueryDSL 5.1.0 (jakarta) |
| DB | MySQL 8 (단일 Master, 데모용) |
| 빌드 | Gradle |
| 코드생성 | Lombok |
| API 문서 | Springdoc OpenAPI 2.8.x |
| 스케줄링 | Spring @Scheduled |

### 1.2 어드민 프론트엔드
| 항목 | 기술 |
|------|------|
| 프레임워크 | Vue 3 (Composition API, script setup) |
| UI 라이브러리 | CoreUI Vue 4.x |
| 상태관리 | Pinia |
| HTTP | Axios (Send.js 래퍼 패턴) |
| 빌드 | Vue CLI 5.x |
| 스타일 | SASS (scoped) |

### 1.3 유저 프론트엔드
| 항목 | 기술 |
|------|------|
| 프레임워크 | Vue 3 + Vite SPA |
| 디자인 시스템 | marketbang.kr 디자인 토큰 적용 (버건디/라임/그레이스케일) |
| 폰트 | Pretendard + Cafe24Ssurround |
| 레이아웃 | 모바일 우선 (viewport-fit=cover) |
| 스타일 | CSS Variables 기반 |

### 1.4 배포
- 로컬 개발 환경에서만 실행 (localhost)
- 백엔드: localhost:8888
- 어드민 프론트: localhost:8081 (추정)
- 유저 프론트: localhost:8080 (추정)

---

## 2. 기능 요구사항 (Functional Requirements)

### 2.1 어드민 기능

#### FR-A01: 공구 등록
- 마켓뱅 상품 URL 입력 시 Mock 데이터로 상품 정보 자동 채움
- 자동 연동 필드: 상품명, 대표 이미지, 상품 설명, 수입사명, 정가
- 수동 입력 필드: 공구 공급가, 목표 수량, 최대 모집 수량, 1인 최소/최대 수량, 모집 시작/종료일시, 배송 예정일

#### FR-A02: 공구 목록 조회
- 상태별 필터: 모집중/확정/자동취소/수동취소/배송준비/완료
- 기본 정보 표시: 제목, 상태, 현재 참여수/목표수, 마감일

#### FR-A03: 공구 상세 조회
- 실시간 참여 현황 (참여자 목록, 수량, 시간)
- 공구 기본 정보 전체

#### FR-A04: 공구 수정
- 모집 중 상태에서만 수정 가능
- 수정 가능 필드: 제목, 가격, 수량, 기간

#### FR-A05: 공구 수동 종료/취소
- 운영자가 수동으로 공구 취소 가능
- 확정 후 "배송 완료" 처리 가능

### 2.2 유저 기능

#### FR-U01: 공구 목록 (진행중)
- 카드형 UI로 진행 중인 공구 표시
- 각 카드: 상품 이미지, 상품명, 가격(정가→공구가), 달성률, 마감 시간

#### FR-U02: 공구 상세
- 상품 이미지, 상품명, 수입사명
- 가격 표시: 정가 대비 공구가 할인율
- 프로그레스 바: 목표 수량 대비 현재 참여 수량 (백분율)
- 카운트다운 타이머: 마감까지 남은 시간 (일/시/분)
- 참여자 수 표시
- 최근 참여자 알림: "○○와인바님 N분 전 참여" (마스킹)

#### FR-U03: 참여 신청
- 수량 선택: -/+ 버튼 (최소/최대 수량 검증)
- 이메일 입력 (데모용 인증)
- 예상 결제 금액 표시
- 참여 확인 → "데모 결제 처리" 안내 메시지

#### FR-U04: 참여 취소
- 공구 마감 전까지 자유롭게 취소 가능
- 취소 시 참여 수량 차감, 참여자 수 -1

#### FR-U05: 내 참여 내역
- 이메일 기준으로 참여한 공구 목록 조회
- 각 공구의 상태(참여중/확정/취소) 확인

#### FR-U06: 공구 상태 표시
- 확정 시: "공구 확정! 결제가 진행됩니다" 안내
- 취소 시: "목표 미달로 공구가 취소되었습니다. 결제되지 않았습니다" 안내

### 2.3 시스템 기능

#### FR-S01: 마감 자동 처리
- Spring @Scheduled (1분 간격)
- 모집 종료일시 도달 시:
  - 현재수량 >= 목표수량 → 상태 "확정"(CONFIRMED)
  - 현재수량 < 목표수량 → 상태 "자동취소"(AUTO_CANCELLED)

#### FR-S02: 실시간 현황 업데이트
- 폴링 방식 (5초 간격)
- 참여 현황, 프로그레스 바, 카운트다운 자동 갱신

#### FR-S03: 동시성 처리
- 목표/최대 수량 초과 방지 (선착순)
- 마감 직전 참여: 서버 시각 기준 판단

---

## 3. 비기능 요구사항 (Non-Functional Requirements)

### 3.1 성능
- 폴링 간격: 5초
- 스케줄러 간격: 1분
- 동시 참여 처리: 비관적 락 또는 낙관적 락으로 동시성 제어

### 3.2 데이터
- 단일 MySQL DB (로컬)
- 소프트 삭제 패턴 (useYn = N)
- UUID 기반 공구/참여 ID

### 3.3 인증
- 어드민: 간단한 ID/PW 로그인 (세션 또는 JWT)
- 유저: 이메일만 입력 (세션 스토리지/쿠키 기반)
- 복잡한 인증 시스템 불필요

### 3.4 UX
- 유저 페이지: 모바일 우선 디자인
- 어드민 페이지: 데스크톱 기준 CoreUI
- 마켓뱅 디자인 토큰 적용 (버건디 #911054, 라임 #7fe816, 본문 #222)

---

## 4. 데이터 모델

### 4.1 GroupBuy (공동구매)
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| id | UUID/Long | O | PK |
| title | String | O | 공구 제목 |
| product_url | String | O | 마켓뱅 상품 URL |
| product_name | String | O | 상품명 |
| product_image_url | String | O | 대표 이미지 |
| product_description | String | X | 상품 설명 |
| importer_name | String | X | 수입사명 |
| original_price | Integer | O | 정가 |
| groupbuy_price | Integer | O | 공구 공급가 |
| target_quantity | Integer | O | 목표 수량 |
| max_quantity | Integer | X | 최대 모집 수량 (null=무제한) |
| min_per_user | Integer | O | 1인 최소 수량 (기본 1) |
| max_per_user | Integer | X | 1인 최대 수량 (null=제한없음) |
| start_at | LocalDateTime | O | 모집 시작일시 |
| end_at | LocalDateTime | O | 모집 종료일시 |
| delivery_date | LocalDate | X | 배송 예정일 |
| status | Enum | O | 모집중/확정/자동취소/수동취소/배송준비/완료 |
| current_quantity | Integer | O | 현재 참여 총 수량 |
| participant_count | Integer | O | 참여자 수 |
| created_at | LocalDateTime | O | 생성일시 |
| updated_at | LocalDateTime | O | 수정일시 |

### 4.2 Participation (참여)
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| id | UUID/Long | O | PK |
| groupbuy_id | Long | O | FK → GroupBuy |
| user_email | String | O | 참여자 이메일 |
| user_display_name | String | O | 소매처명 |
| quantity | Integer | O | 참여 수량 |
| expected_amount | Integer | O | 예상 결제 금액 |
| status | Enum | O | 참여중/취소/확정 |
| created_at | LocalDateTime | O | 참여일시 |
| cancelled_at | LocalDateTime | X | 취소일시 |

### 4.3 AdminUser (어드민 사용자 - 간소화)
| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| id | Long | O | PK |
| user_id | String | O | 로그인 ID |
| user_pwd | String | O | 비밀번호 (인코딩) |
| user_name | String | O | 이름 |
| use_yn | Enum | O | 사용 여부 |

### 4.4 상태 머신 (GroupBuy.status)
```
RECRUITING (모집중) → CONFIRMED (확정) → PREPARING (배송준비) → COMPLETED (완료)
                   → AUTO_CANCELLED (자동취소)
                   → MANUAL_CANCELLED (수동취소)
```

---

## 5. 제외 사항 (Out of Scope)

PRD 및 constraints.md 기준:
- 실제 결제 처리 (PG사 연동) — 데모 메시지로 대체
- 배송 트래킹 — 상태 표시만
- 알림 (이메일/SMS/푸시) — 화면 내 안내만
- 티어 가격 구조 — 제외
- 복잡한 인증 (OAuth, SNS 로그인, 2FA)
- 이미지 리사이징/최적화
- 데이터 분석/대시보드/매출 리포트
- 재고 관리, 직원 관리, 예약 시스템
- 고객 리뷰, 다국어, 외부 연동

---

## 6. API 엔드포인트 설계 (예상)

### 어드민 API
```
POST   /api/admin/v1/groupbuy           — 공구 등록
GET    /api/admin/v1/groupbuy/list      — 공구 목록
GET    /api/admin/v1/groupbuy/{id}      — 공구 상세
PUT    /api/admin/v1/groupbuy/{id}      — 공구 수정
POST   /api/admin/v1/groupbuy/{id}/cancel — 공구 수동 취소
POST   /api/admin/v1/groupbuy/{id}/complete — 배송 완료 처리
POST   /api/admin/v1/user/login         — 어드민 로그인
GET    /api/admin/v1/groupbuy/{id}/participants — 참여자 목록
```

### 유저(공개) API
```
GET    /api/buyer/v1/public/groupbuy/list     — 진행중 공구 목록
GET    /api/buyer/v1/public/groupbuy/{id}     — 공구 상세
POST   /api/buyer/v1/public/groupbuy/{id}/join — 참여 신청
POST   /api/buyer/v1/public/groupbuy/{id}/cancel — 참여 취소
GET    /api/buyer/v1/public/groupbuy/my?email={email} — 내 참여 내역
```

### Mock API
```
GET    /api/common/v1/product/fetch?url={url} — 마켓뱅 상품 정보 Mock 조회
```

---

## 7. 화면 목록

### 어드민 (5개 화면)
1. 로그인
2. 공구 목록 (상태 탭 필터)
3. 공구 등록 (URL 입력 → 자동채움 → 폼)
4. 공구 상세 (참여 현황 포함)
5. 공구 수정

### 유저 (4개 화면)
1. 공구 목록 (카드형, 진행중)
2. 공구 상세 (프로그레스바, 카운트다운, 참여하기)
3. 참여 완료 확인
4. 내 참여 내역 (이메일 조회)

---

## 8. Extension Configuration

| Extension | Enabled | Decided At |
|---|---|---|
| Security Baseline | No | Requirements Analysis |
| Property-Based Testing | No | Requirements Analysis |
