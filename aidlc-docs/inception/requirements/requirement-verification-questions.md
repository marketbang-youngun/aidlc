# 요구사항 확인 질문

제공된 요구사항 문서를 분석한 결과, 아래 항목들에 대한 추가 확인이 필요합니다.
각 질문의 [Answer]: 태그 뒤에 선택지 문자를 입력해 주세요.

## Question 1
백엔드 기술 스택으로 어떤 언어/프레임워크를 사용할까요?

A) Node.js + Express
B) Node.js + NestJS
C) Python + FastAPI
D) Java + Spring Boot
E) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 2
프론트엔드 기술 스택으로 어떤 프레임워크를 사용할까요?

A) React (Create React App 또는 Vite)
B) Next.js
C) Vue.js
D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 3
데이터베이스로 어떤 것을 사용할까요?

A) PostgreSQL
B) MySQL
C) DynamoDB (AWS NoSQL)
D) MongoDB
E) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 4
배포 대상 환경은 어디인가요?

A) AWS (EC2, ECS, Lambda 등)
B) 로컬 서버 / Docker Compose (개발 환경 우선)
C) AWS Serverless (Lambda + API Gateway + DynamoDB)
D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 5
고객용 UI와 관리자용 UI를 어떻게 구성할까요?

A) 하나의 프론트엔드 프로젝트에서 경로(Route)로 분리
B) 별도의 프론트엔드 프로젝트로 분리 (고객용, 관리자용 각각)
C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 6
매장(Store) 데이터 구조에 대해 확인합니다. 이 시스템은 단일 매장용인가요, 멀티 매장(SaaS)용인가요?

A) 단일 매장 전용 (하나의 매장만 운영)
B) 멀티 매장 지원 (여러 매장이 각각 독립적으로 사용)
C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 7
관리자 계정 관리 방식은 어떻게 할까요?

A) 시스템에 관리자 계정을 사전 등록 (시드 데이터 또는 CLI로 생성)
B) 관리자 회원가입 기능 포함
C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 8
메뉴 이미지 관리 방식은 어떻게 할까요?

A) 외부 이미지 URL만 입력 (이미지 업로드 없음)
B) 이미지 파일 업로드 기능 포함 (서버 또는 S3 저장)
C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 9
테스트 전략은 어떤 수준을 원하시나요?

A) 단위 테스트 (Unit Test) 위주
B) 단위 테스트 + API 통합 테스트
C) 단위 테스트 + 통합 테스트 + E2E 테스트
D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 10: Security Extensions
이 프로젝트에 보안 확장 규칙을 적용할까요?

A) Yes — 모든 보안 규칙을 blocking 제약조건으로 적용 (프로덕션 수준 애플리케이션에 권장)
B) No — 보안 규칙 건너뛰기 (PoC, 프로토타입, 실험 프로젝트에 적합)
C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 11: Property-Based Testing Extension
Property-Based Testing (PBT) 규칙을 적용할까요?

A) Yes — 모든 PBT 규칙을 blocking 제약조건으로 적용 (비즈니스 로직, 데이터 변환, 직렬화, 상태 컴포넌트가 있는 프로젝트에 권장)
B) Partial — 순수 함수와 직렬화 round-trip에만 PBT 규칙 적용 (알고리즘 복잡도가 제한적인 프로젝트에 적합)
C) No — PBT 규칙 건너뛰기 (단순 CRUD, UI 전용, 얇은 통합 레이어에 적합)
D) Other (please describe after [Answer]: tag below)

[Answer]: 
