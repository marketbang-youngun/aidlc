# Unit of Work Definitions

## Unit Overview

| Unit | 이름 | 담당자 | 폴더 | 기술 |
|------|------|--------|------|------|
| **Unit 1** | Backend API | 백엔드 개발자 | `backend/` | Spring Boot 3 + Java 17 + MySQL |
| **Unit 2** | Frontend | 프론트 개발자 | `admin-front/` + `user-front/` | Vue 3 + CoreUI / Vue 3 + Vite |

---

## Unit 1: Backend API

### 책임
- REST API 전체 제공 (어드민 9 + 유저 6 + 공통 1 엔드포인트)
- 비즈니스 로직 (공구 CRUD, 참여 처리, 동시성 제어)
- 데이터 영속성 (MySQL, JPA + QueryDSL)
- 인증 (어드민 JWT Cookie)
- 마감 자동 처리 (스케줄러)
- API 문서 (Swagger/OpenAPI)

### 산출물
- Spring Boot 프로젝트 (`backend/`)
- DDL 스크립트 (`backend/src/main/resources/schema.sql`)
- application.yml 설정
- Swagger API 문서 (자동 생성)

### 착수 조건
- 없음 (가장 먼저 착수)

### 완료 기준
- 전체 API 엔드포인트 동작
- Swagger UI에서 API 테스트 가능
- 스케줄러 동작 확인
- Mock 상품 데이터 응답 확인

---

## Unit 2: Frontend (Admin + User)

### 책임
- 어드민 관리 화면 (공구 등록/수정/목록/상세/취소)
- 유저 참여 화면 (공구 목록/상세/참여/취소/내역)
- UI/UX (마켓뱅 디자인 시스템 적용)
- API 연동 (Backend API 호출)
- 폴링 (5초 간격 현황 갱신)

### 산출물
- Admin Front 프로젝트 (`admin-front/`)
- User Front 프로젝트 (`user-front/`)

### 착수 조건
- Backend API 스펙(Swagger) 확정 후 본격 개발
- API 미완성 시에도 Mock 데이터로 UI 작업 선행 가능

### 완료 기준
- 어드민: 로그인 → 공구 등록 → 목록 확인 → 상세 → 수정/취소 플로우 동작
- 유저: 목록 → 상세 → 참여 → 폴링 갱신 → 취소 → 내역 조회 플로우 동작
- 마켓뱅 디자인 토큰 적용 (유저 페이지)

---

## Code Organization (Greenfield)

```
marketbang2/                    # 모노레포 루트
├── backend/                    # Unit 1: Spring Boot
│   ├── build.gradle
│   ├── settings.gradle
│   └── src/
│       ├── main/
│       │   ├── java/com/threelabs/
│       │   │   ├── config/
│       │   │   ├── controller/
│       │   │   ├── service/
│       │   │   ├── repository/
│       │   │   ├── entity/
│       │   │   ├── dto/
│       │   │   ├── enumration/
│       │   │   ├── error/
│       │   │   ├── schedule/
│       │   │   └── util/
│       │   └── resources/
│       │       ├── application.yml
│       │       └── schema.sql
│       └── test/
├── admin-front/                # Unit 2-A: Admin Vue CLI
│   ├── package.json
│   └── src/
│       ├── api/
│       ├── views/
│       ├── components/
│       ├── pinia/
│       ├── router/
│       └── main.js
├── user-front/                 # Unit 2-B: User Vite
│   ├── package.json
│   └── src/
│       ├── api/
│       ├── views/
│       ├── components/
│       ├── composables/
│       ├── router/
│       ├── assets/
│       └── main.js
├── requirements/               # 요구사항 문서
├── aidlc-docs/                 # AI-DLC 문서
├── .gitignore
└── CLAUDE.md
```

---

## 병렬 작업 전략

```
Week 1:
  Backend 개발자: 프로젝트 셋업 + Entity/Repository + API 구현
  프론트 개발자: 프로젝트 셋업 + 라우팅 + UI 컴포넌트 (Mock 데이터)

Week 1 중반: Backend Swagger 공유
  프론트 개발자: API 연동 시작

Week 2:
  Backend 개발자: 스케줄러 + 동시성 + 엣지케이스
  프론트 개발자: 폴링 + 카운트다운 + 통합 테스트
```
