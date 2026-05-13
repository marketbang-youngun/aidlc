# Unit of Work — Dependency Matrix

## Inter-Unit Dependencies

```
+-------------------+          API (HTTP/JSON)          +-------------------+
|                   | ◄──────────────────────────────── |                   |
|   Unit 1:         |                                   |   Unit 2:         |
|   Backend API     |                                   |   Frontend        |
|   (:8888)         | ────────────────────────────────► |   (:8080/:8081)   |
|                   |          JSON Responses            |                   |
+-------------------+                                   +-------------------+
        │
        │ JDBC
        ▼
+-------------------+
|    MySQL DB       |
|    (:3306)        |
+-------------------+
```

## Dependency Direction

| From | To | Type | 설명 |
|------|----|------|------|
| Unit 2 (Frontend) | Unit 1 (Backend) | HTTP REST | API 호출 (CORS) |
| Unit 1 (Backend) | MySQL | JDBC | 데이터 영속성 |

**핵심**: Unit 2 → Unit 1 단방향 의존. Backend는 Frontend에 의존하지 않음.

---

## 공유 리소스

| 리소스 | 소유 Unit | 사용 Unit | 동기화 방법 |
|--------|-----------|-----------|-------------|
| API 스펙 (Swagger) | Unit 1 | Unit 2 | Swagger UI URL 공유 |
| DB Schema | Unit 1 | Unit 1 only | schema.sql |
| 디자인 토큰 | Unit 2 | Unit 2 only | CSS Variables |

---

## 통신 패턴

### Admin Front → Backend
```
요청: POST/GET/PUT + JWT Cookie (인증)
응답: { code: 0, message: "", data: {...}, isLogin: true }
CORS Origin: http://localhost:8081
```

### User Front → Backend
```
요청: GET/POST (인증 없음, email 파라미터)
응답: { code: 0, message: "", data: {...}, isLogin: false }
CORS Origin: http://localhost:8080
폴링: GET /api/buyer/v1/public/groupbuy/{id}/status (5초)
```

---

## 구현 순서 제약

| 순서 | 항목 | 이유 |
|------|------|------|
| 1 | Backend: Entity + Repository | DB 기반 |
| 2 | Backend: Service + Controller | API 제공 |
| 3 | Backend: Swagger 확인 | 프론트 연동 기준 |
| 4 | Frontend: API 연동 | Backend API 필요 |

**단, 프론트는 UI/컴포넌트 작업을 API 없이 선행 가능 (Mock 데이터 사용)**

---

## 독립 개발 가능 영역

### Backend (API 없이 독립 개발 가능)
- Entity, Repository, Service, Controller 전체
- 스케줄러
- Security 설정
- 단위 테스트

### Frontend (Backend 없이 독립 개발 가능)
- 라우팅 설정
- UI 컴포넌트 (레이아웃, 카드, 폼)
- CSS/디자인 토큰 적용
- 카운트다운 타이머, 프로그레스바 (정적 데이터)
- Pinia 스토어 구조
