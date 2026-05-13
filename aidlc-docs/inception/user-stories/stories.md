# User Stories

---

## Epic 1: 공구 관리 (어드민)

### AS-01: 어드민 로그인
**As a** 공동구매 운영자 (김수연)
**I want to** ID와 비밀번호로 어드민 시스템에 로그인하고 싶다
**So that** 권한 있는 사용자만 공구를 관리할 수 있다

**Acceptance Criteria:**
- [ ] Given 올바른 ID/PW 입력 When 로그인 버튼 클릭 Then 공구 목록 페이지로 이동
- [ ] Given 잘못된 ID/PW 입력 When 로그인 버튼 클릭 Then 에러 메시지 표시 "아이디 또는 비밀번호가 일치하지 않습니다"
- [ ] Given 로그인하지 않은 상태 When 어드민 페이지 접근 Then 로그인 페이지로 리다이렉트

**Priority**: Must
**Dependencies**: 없음

---

### AS-02: 공구 등록 — 상품 정보 연동
**As a** 공동구매 운영자 (김수연)
**I want to** 마켓뱅 상품 URL을 입력하면 상품 기본 정보가 자동으로 채워지길 원한다
**So that** 공구 등록 시간을 단축하고 입력 실수를 줄인다

**Acceptance Criteria:**
- [ ] Given 공구 등록 화면 When 마켓뱅 상품 URL 입력 후 "불러오기" 클릭 Then 상품명, 이미지, 설명, 수입사명, 정가가 자동 채움 (Mock 데이터)
- [ ] Given 잘못된 형식의 URL 입력 When "불러오기" 클릭 Then 에러 메시지 "올바른 마켓뱅 상품 URL을 입력해주세요"
- [ ] Given 자동 채움된 정보 When 운영자가 확인 Then 수정 가능한 상태로 표시

**Priority**: Must
**Dependencies**: 없음

---

### AS-03: 공구 등록 — 공구 설정
**As a** 공동구매 운영자 (김수연)
**I want to** 공구 공급가, 목표 수량, 모집 기간을 설정하여 공구를 오픈하고 싶다
**So that** 소매처들이 참여할 수 있는 공구가 활성화된다

**Acceptance Criteria:**
- [ ] Given 상품 정보가 채워진 상태 When 필수 필드(공급가, 목표수량, 시작일시, 종료일시) 입력 후 등록 Then 공구가 생성되고 목록에 표시
- [ ] Given 종료일시 < 시작일시 When 등록 시도 Then 에러 "종료일시는 시작일시 이후여야 합니다"
- [ ] Given 공급가 미입력 When 등록 시도 Then 필수 입력 검증 에러 표시
- [ ] Given 최대모집수량 < 목표수량 When 등록 시도 Then 에러 "최대 모집 수량은 목표 수량 이상이어야 합니다"
- [ ] Given 모집 시작일시가 현재 이후 When 등록 완료 Then 상태는 "모집중(RECRUITING)"으로 시작

**Priority**: Must
**Dependencies**: AS-02

---

### AS-04: 공구 목록 조회
**As a** 공동구매 운영자 (김수연)
**I want to** 전체 공구를 상태별로 필터링하여 목록으로 보고 싶다
**So that** 진행 중/완료/취소된 공구를 효율적으로 관리한다

**Acceptance Criteria:**
- [ ] Given 어드민 로그인 상태 When 공구 목록 페이지 접근 Then 전체 공구가 최신순으로 표시
- [ ] Given 공구 목록 When 상태 탭(모집중/확정/취소/배송준비/완료) 클릭 Then 해당 상태 공구만 필터링
- [ ] Given 공구 목록 항목 When 각 행에 Then 제목, 상태, 현재수량/목표수량, 마감일 표시

**Priority**: Must
**Dependencies**: AS-01

---

### AS-05: 공구 상세 조회 (참여 현황)
**As a** 공동구매 운영자 (김수연)
**I want to** 진행 중인 공구의 참여 현황을 실시간으로 확인하고 싶다
**So that** 목표 달성 여부를 예측하고 필요시 대응할 수 있다

**Acceptance Criteria:**
- [ ] Given 공구 목록에서 항목 클릭 When 상세 페이지 진입 Then 공구 기본 정보 + 참여 현황 표시
- [ ] Given 공구 상세 페이지 When 참여자 목록 영역 Then 참여자 이메일(마스킹), 소매처명, 수량, 참여일시 표시
- [ ] Given 공구 상세 페이지 When 새 참여자 발생 Then 페이지 새로고침 없이 현황 업데이트 (폴링)

**Priority**: Must
**Dependencies**: AS-04

---

### AS-06: 공구 수정
**As a** 공동구매 운영자 (김수연)
**I want to** 모집 중인 공구의 정보를 수정하고 싶다
**So that** 잘못 입력한 정보를 바로잡을 수 있다

**Acceptance Criteria:**
- [ ] Given 모집중(RECRUITING) 상태 공구 When 수정 버튼 클릭 Then 수정 가능 필드가 편집 모드로 전환
- [ ] Given 확정/취소 상태 공구 When 수정 시도 Then 수정 불가 안내 메시지
- [ ] Given 수정 화면 When 변경사항 저장 Then 공구 정보 업데이트, updated_at 갱신

**Priority**: Should
**Dependencies**: AS-05

---

### AS-07: 공구 수동 취소/완료
**As a** 공동구매 운영자 (김수연)
**I want to** 필요 시 공구를 수동으로 취소하거나, 확정된 공구를 배송 완료 처리하고 싶다
**So that** 예외 상황에 유연하게 대응한다

**Acceptance Criteria:**
- [ ] Given 모집중 상태 공구 When "수동 취소" 버튼 클릭 + 확인 Then 상태가 MANUAL_CANCELLED로 변경
- [ ] Given 확정(CONFIRMED) 상태 공구 When "배송준비" 버튼 클릭 Then 상태가 PREPARING으로 변경
- [ ] Given 배송준비(PREPARING) 상태 공구 When "배송 완료" 버튼 클릭 Then 상태가 COMPLETED로 변경
- [ ] Given 취소 또는 완료 상태 공구 When 상태 변경 시도 Then 변경 불가

**Priority**: Must
**Dependencies**: AS-05

---

## Epic 2: 공구 참여 (유저)

### US-01: 진행중 공구 목록 탐색
**As a** 소매처 구매 담당자 (박성진)
**I want to** 현재 진행 중인 공구 상품을 카드형 목록으로 한눈에 보고 싶다
**So that** 관심 있는 공구를 빠르게 찾고 상세를 확인할 수 있다

**Acceptance Criteria:**
- [ ] Given 유저 메인 페이지 접근 When 페이지 로드 Then 모집중(RECRUITING) 상태 공구만 카드형으로 표시
- [ ] Given 카드 하나 When 표시 내용 Then 상품 이미지, 상품명, 정가→공구가(할인율), 달성률 프로그레스바, 마감까지 남은 시간
- [ ] Given 카드 클릭 When 탭 Then 해당 공구 상세 페이지로 이동
- [ ] Given 모집중 공구가 없을 때 When 페이지 로드 Then "현재 진행 중인 공구가 없습니다" 빈 상태 표시

**Priority**: Must
**Dependencies**: 없음

---

### US-02: 공구 상세 보기
**As a** 소매처 구매 담당자 (박성진)
**I want to** 공구의 상세 정보(가격, 목표, 참여 현황, 마감 시간)를 한 화면에서 보고 싶다
**So that** 참여 여부를 판단할 충분한 정보를 얻는다

**Acceptance Criteria:**
- [ ] Given 공구 상세 페이지 When 로드 Then 상품 이미지, 상품명, 수입사명, 정가/공구가/할인율 표시
- [ ] Given 공구 상세 페이지 When 참여 현황 영역 Then 프로그레스 바(목표 대비 현재 수량 %), 참여자 수 표시
- [ ] Given 공구 상세 페이지 When 카운트다운 영역 Then "마감까지 DD일 HH시간 MM분" 실시간 감소
- [ ] Given 공구 상세 페이지 When 최근 참여자 영역 Then "○○와인바님 N분 전 참여" 형태로 마스킹된 최근 참여 표시
- [ ] Given 마감된 공구 상세 페이지 When 로드 Then 참여 버튼 비활성, 상태(확정/취소) 안내 메시지 표시

**Priority**: Must
**Dependencies**: US-01

---

### US-03: 공구 참여 신청
**As a** 소매처 구매 담당자 (박성진)
**I want to** 원하는 수량을 선택하고 이메일을 입력하여 공구에 참여하고 싶다
**So that** 공구가 확정되면 할인 가격으로 상품을 구매할 수 있다

**Acceptance Criteria:**
- [ ] Given 공구 상세 페이지 When 수량 -/+ 버튼 조작 Then 최소~최대 수량 범위 내에서 조절, 예상 결제 금액 실시간 계산
- [ ] Given 수량 선택 완료 When "참여하기" 버튼 클릭 Then 이메일 + 소매처명 입력 폼 표시
- [ ] Given 이메일 + 소매처명 입력 When "참여 확인" 클릭 Then 참여 완료, "결제는 데모용으로 처리되었습니다" 안내 메시지
- [ ] Given 참여 완료 When 화면 전환 Then 참여 내역 + 현재 참여 현황으로 이동
- [ ] Given 최대 모집 수량에 도달한 공구 When 참여 시도 Then "마감되었습니다" 안내
- [ ] Given 이미 참여한 이메일 When 동일 공구 재참여 시도 Then "이미 참여하셨습니다" 안내 또는 수량 추가 처리

**Priority**: Must
**Dependencies**: US-02

---

### US-04: 참여 취소
**As a** 소매처 구매 담당자 (박성진)
**I want to** 참여 후 마음이 바뀌면 마감 전까지 취소할 수 있길 원한다
**So that** 부담 없이 참여를 결정할 수 있다

**Acceptance Criteria:**
- [ ] Given 참여중 상태이고 공구가 모집중 When "참여 취소" 버튼 클릭 Then 취소 확인 다이얼로그 표시
- [ ] Given 취소 확인 When "취소하기" 클릭 Then 참여 상태 → 취소, 공구 현재 수량 차감, 참여자 수 -1
- [ ] Given 공구가 마감(확정/취소)된 이후 When 참여 취소 시도 Then "마감 후에는 취소할 수 없습니다" 안내

**Priority**: Must
**Dependencies**: US-03

---

### US-05: 내 참여 내역 조회
**As a** 소매처 구매 담당자 (박성진)
**I want to** 이메일로 내가 참여한 공구 목록과 각 공구의 상태를 확인하고 싶다
**So that** 참여한 공구의 확정/취소 결과를 파악한다

**Acceptance Criteria:**
- [ ] Given 내 참여 내역 페이지 When 이메일 입력 후 조회 Then 해당 이메일로 참여한 공구 목록 표시
- [ ] Given 참여 내역 목록 When 각 항목에 Then 공구 제목, 참여 수량, 예상 금액, 참여 상태(참여중/확정/취소) 표시
- [ ] Given 참여 내역에서 공구 클릭 When 탭 Then 해당 공구 상세 페이지로 이동
- [ ] Given 해당 이메일로 참여한 내역이 없을 때 When 조회 Then "참여 내역이 없습니다" 빈 상태 표시

**Priority**: Should
**Dependencies**: US-03

---

### US-06: 공구 확정/취소 결과 확인
**As a** 소매처 구매 담당자 (박성진)
**I want to** 공구가 확정되었는지 취소되었는지 명확히 알고 싶다
**So that** 결제 진행 여부를 파악하고 발주 계획을 세울 수 있다

**Acceptance Criteria:**
- [ ] Given 참여한 공구가 확정(CONFIRMED)됨 When 상세 페이지 또는 내역에서 확인 Then "공구 확정! 결제가 진행됩니다" 안내 + 배송 예정일 표시
- [ ] Given 참여한 공구가 자동취소(AUTO_CANCELLED)됨 When 확인 Then "목표 미달로 공구가 취소되었습니다. 결제되지 않았습니다" 안내
- [ ] Given 참여한 공구가 수동취소(MANUAL_CANCELLED)됨 When 확인 Then "운영자에 의해 공구가 종료되었습니다. 결제되지 않았습니다" 안내

**Priority**: Must
**Dependencies**: US-02, SS-01

---

## Epic 3: 시스템 자동 처리

### SS-01: 마감 자동 확정/취소
**As a** 시스템
**I want to** 모집 종료 시점에 목표 달성 여부에 따라 자동으로 상태를 변경한다
**So that** 운영자 개입 없이 공구 라이프사이클이 완결된다

**Acceptance Criteria:**
- [ ] Given 모집중 공구의 end_at이 현재 시각 이전 AND current_quantity >= target_quantity When 스케줄러 실행 Then 상태 → CONFIRMED, 참여자 status → 확정
- [ ] Given 모집중 공구의 end_at이 현재 시각 이전 AND current_quantity < target_quantity When 스케줄러 실행 Then 상태 → AUTO_CANCELLED
- [ ] Given 스케줄러 주기 When 매 1분 Then 마감 대상 공구 체크 및 처리

**Priority**: Must
**Dependencies**: 없음

---

### SS-02: 실시간 참여 현황 폴링
**As a** 시스템
**I want to** 유저 페이지에서 5초 간격으로 참여 현황을 갱신한다
**So that** 소매처가 최신 참여 상태를 실시간에 가깝게 확인할 수 있다

**Acceptance Criteria:**
- [ ] Given 공구 상세 페이지 열림 When 5초 경과 Then 서버에서 최신 참여 현황(수량, 참여자 수, 최근 참여자) 조회
- [ ] Given 폴링 응답 수신 When 데이터 변경 있음 Then 프로그레스바, 참여자 수, 최근 참여자 UI 갱신
- [ ] Given 공구 상태가 모집중이 아님 When 감지 Then 폴링 중단, 최종 상태 표시

**Priority**: Should
**Dependencies**: US-02

---

### SS-03: 동시성 제어 (초과 참여 방지)
**As a** 시스템
**I want to** 동시에 여러 참여 요청이 들어와도 최대 모집 수량을 초과하지 않도록 한다
**So that** 데이터 정합성이 유지되고 초과 참여를 방지한다

**Acceptance Criteria:**
- [ ] Given max_quantity가 설정된 공구 When 동시에 여러 참여 요청 Then 합산 수량이 max_quantity를 초과하지 않음
- [ ] Given 초과 발생 시도 When 처리 Then 후순위 요청에 "마감되었습니다" 응답
- [ ] Given 마감 시각 직전 참여 When 서버 시각 기준 Then end_at 이전이면 허용, 이후면 거부

**Priority**: Must
**Dependencies**: 없음

---

## Story Map Summary

| Priority | Stories |
|----------|---------|
| **Must** | AS-01, AS-02, AS-03, AS-04, AS-05, AS-07, US-01, US-02, US-03, US-04, US-06, SS-01, SS-03 |
| **Should** | AS-06, US-05, SS-02 |

**총 스토리 수**: 16개 (Must: 13, Should: 3)
