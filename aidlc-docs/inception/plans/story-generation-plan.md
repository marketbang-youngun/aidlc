# Story Generation Plan

## Context
- PRD에 핵심 사용자 스토리가 이미 정의됨 (AS-1~4, US-1~5)
- 요구사항 문서에서 기능 명세와 데이터 모델이 확정됨
- 두 페르소나: 어드민(마켓뱅 운영자), 유저(소매처)

## Approach
- **Feature-Based + Persona-Based Hybrid**: 페르소나별로 그룹핑하되 기능 단위로 스토리 작성
- PRD 기반 스토리를 INVEST 기준으로 재구성
- 각 스토리에 Acceptance Criteria 추가

## Generation Steps

### Phase 1: Personas
- [x] 어드민 페르소나 정의 (마켓뱅 공동구매 운영자)
- [x] 유저 페르소나 정의 (소매처 구매 담당자)

### Phase 2: Admin Stories (Epic: 공구 관리)
- [x] 공구 등록 스토리 (상품 URL Mock 연동 + 폼 설정)
- [x] 공구 목록 조회 스토리 (상태별 필터)
- [x] 공구 상세 조회 스토리 (실시간 참여 현황)
- [x] 공구 수정 스토리
- [x] 공구 수동 취소/완료 처리 스토리
- [x] 어드민 로그인 스토리

### Phase 3: User Stories (Epic: 공구 참여)
- [x] 공구 목록 탐색 스토리 (카드형, 진행중)
- [x] 공구 상세 보기 스토리 (프로그레스바, 카운트다운, 소셜 증거)
- [x] 공구 참여 신청 스토리 (수량 선택, 이메일 입력)
- [x] 참여 취소 스토리
- [x] 내 참여 내역 조회 스토리

### Phase 4: System Stories (Epic: 자동 처리)
- [x] 마감 자동 처리 스토리 (확정/취소)
- [x] 실시간 현황 폴링 스토리
- [x] 동시성 제어 스토리 (선착순 초과 방지)

### Phase 5: Artifacts
- [x] stories.md 생성 (전체 스토리 + Acceptance Criteria)
- [x] personas.md 생성 (페르소나 정의)
- [x] INVEST 기준 검증

## Story Format Template
```
### [Story ID]: [제목]
**As a** [페르소나]
**I want to** [원하는 행동]
**So that** [기대 가치]

**Acceptance Criteria:**
- [ ] Given [조건] When [행동] Then [결과]
- [ ] ...

**Priority**: Must/Should/Nice
**Dependencies**: [의존 스토리 ID]
```
