# Requirements Verification Questions

PRD 및 기술 가이드를 검토했습니다. 아래 질문에 답변해 주시면 요구사항을 확정하겠습니다.
각 질문의 [Answer]: 뒤에 선택지 알파벳을 입력해주세요.

---

## Question 1
백엔드 기술 스택은 thresslabs-style-guide.md에 기술된 Spring Boot 3 + Java 17 구조를 그대로 따르시겠습니까?

A) 예 — Spring Boot 3 + Java 17 + JPA + QueryDSL + MySQL (가이드 그대로)
B) Spring Boot 3 + Java 17이지만 MyBatis 위주로 (QueryDSL 미사용)
C) 다른 백엔드 기술 스택 사용 (Node.js, Python 등)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 2
프론트엔드는 어드민(관리자)과 유저(소매처) 페이지 모두 구현하시겠습니까?

A) 어드민 + 유저 모두 구현 (Vue 3 기반, 어드민은 CoreUI)
B) 유저 페이지만 구현 (어드민은 Swagger/API로 대체)
C) 어드민 페이지만 구현 (유저 API만 제공)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 3
유저(소매처) 프론트엔드 기술 스택은 무엇으로 하시겠습니까?

A) Vue 3 + Vite SPA (marketbang_design_system-user.md의 디자인 시스템 적용, 모바일 우선)
B) Vue 3 + Nuxt.js (SSR)
C) React + Next.js
D) 순수 HTML/CSS/JS (가볍게)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 4
마켓뱅 상품 URL 연동(어드민에서 상품 정보 자동 불러오기)은 어떻게 처리하시겠습니까?

A) Mock 데이터로 대체 (실제 마켓뱅 API 호출 없이 더미 상품 정보 사용)
B) 실제 마켓뱅 API 연동 (기존 백엔드에 상품 조회 API가 있다고 가정)
C) 어드민에서 수동으로 모든 상품 정보 직접 입력 (URL 연동 기능 제외)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 5
데이터베이스 구조는 어떻게 하시겠습니까?

A) 단일 MySQL DB (Master만, 데모용으로 간단히)
B) Master/Replica 분리 (thresslabs-style-guide.md 구조 그대로)
C) H2 인메모리 DB (로컬 데모 전용, 재시작 시 데이터 초기화)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 6
인증 시스템은 어떻게 구현하시겠습니까?

A) PRD대로 데모용 간소화 — 유저는 이메일만 입력, 어드민은 간단한 ID/PW 로그인
B) 기존 마켓뱅 JWT Cookie 방식 그대로 적용 (thresslabs-style-guide.md 참조)
C) 인증 없이 — 어드민/유저 모두 로그인 없이 사용 (순수 데모)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 7
티어 가격 구조(수량에 따른 단계별 가격 인하) 기능을 이번 빌드에 포함하시겠습니까?

A) 포함 — 티어 가격 구조 완전 구현
B) 데이터 모델만 포함하되, UI/로직은 단순 단일 가격으로 처리 (향후 확장 대비)
C) 제외 — 단일 공구 가격만 사용
X) Other (please describe after [Answer]: tag below)

[Answer]: c

---

## Question 8
실시간 참여 현황 업데이트 방식은 어떻게 하시겠습니까?

A) 폴링 방식 (5초 간격으로 서버 조회) — PRD 권장
B) WebSocket (실시간 양방향 통신)
C) 수동 새로고침만 (자동 업데이트 없음)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 9
마감 자동 처리(스케줄러)는 어떻게 구현하시겠습니까?

A) Spring @Scheduled 사용 (1분 간격으로 마감 대상 공구 확인 후 자동 처리)
B) 유저 접근 시점에 체크 (Lazy evaluation — 마감 시간이 지난 공구에 접근하면 그때 처리)
C) 수동 처리만 (어드민이 직접 확정/취소 버튼 클릭)
X) Other (please describe after [Answer]: tag below)

[Answer]: a

---

## Question 10
배포 환경은 어떻게 계획하고 계십니까?

A) AWS 배포 (EC2 + RDS + S3/CloudFront) — 운영과 유사한 환경
B) 로컬 개발 환경에서만 실행 (배포 없음, localhost)
C) Docker Compose로 로컬 + 필요 시 클라우드 배포 가능하도록
X) Other (please describe after [Answer]: tag below)

[Answer]: b

---

## Question 11: Security Extensions
이 프로젝트에 보안 확장 규칙을 적용하시겠습니까?

A) 예 — 모든 SECURITY 규칙을 블로킹 제약으로 적용 (프로덕션 수준 애플리케이션에 권장)
B) 아니오 — 모든 SECURITY 규칙 건너뜀 (PoC, 프로토타입, 실험적 프로젝트에 적합)
X) Other (please describe after [Answer]: tag below)

[Answer]: b

---

## Question 12: Property-Based Testing Extension
이 프로젝트에 속성 기반 테스트(Property-Based Testing) 규칙을 적용하시겠습니까?

A) 예 — 모든 PBT 규칙을 블로킹 제약으로 적용 (비즈니스 로직, 데이터 변환, 직렬화, 상태 관리 컴포넌트가 있는 프로젝트에 권장)
B) 부분 적용 — 순수 함수와 직렬화 round-trip에만 PBT 규칙 적용 (알고리즘 복잡도가 제한적인 프로젝트에 적합)
C) 아니오 — 모든 PBT 규칙 건너뜀 (간단한 CRUD 애플리케이션, UI 전용 프로젝트에 적합)
X) Other (please describe after [Answer]: tag below)

[Answer]: c

---

**모든 질문에 답변을 완료하시면 알려주세요.**
