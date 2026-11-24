# Massac 작업 목록 (MVP1 / MVP2)

작성: 2026-11-24 / 기준: `docs/domain.md`, `docs/erd.md` v0.2
전제 [가설]: MVP1은 백엔드 API만 만들고, 화면 대신 Swagger UI로 시연한다.

MVP1 목표: 장애 사례 3개를 인수 테스트로 증명하는 한 흐름을 가장 빨리 완성한다.
신청 입력 → 교육 확정 → 교구 예약 → 강사 배정 → 변경 → 영향 분석 → Task → 예외 대시보드

---

## MVP1

순서대로 진행한다. 각 단계는 PR 1개 단위로 끝내는 것을 권한다.

### 0. 새 레포 셋업
- [x] 새 레포 생성 (사용자), Spring Boot 프로젝트 생성 (Java 17, Gradle)
- [x] docker-compose (MySQL 8, Redis 7)
- [x] Flyway 도입, `ddl-auto=validate`
- [x] Testcontainers 테스트 기반 (클론 후 `./gradlew test` 바로 통과)
- [x] GitHub Actions CI (빌드·테스트), README 배지
- [x] `CLAUDE.md`, `docs/domain.md`, `docs/erd.md`, `docs/backlog.md` 추가
- 완료 기준: 빈 프로젝트에서 `docker compose up -d && ./gradlew test` 통과

### 1. 공통 기반·인증·권한 (새로 작성)
- [ ] 공통 예외 처리 (ErrorCode, CustomException, GlobalExceptionHandler), 생성·수정 시각 공통 엔티티, JPA Auditing
- [ ] JWT 발급·검증, Redis 리프레시 토큰 저장소, SecurityConfig
- [ ] users 확장 (name, phone, email, status), basic_auth
- [ ] 로그인 / 토큰 재발급 / 로그아웃 API
- [ ] 계정 생성: 운영자가 신청자·강사 계정 생성 (자가 회원가입은 MVP2)
- [ ] role, permission, user_role, role_permission + 초기 데이터 (ADMIN은 전체 권한)
- [ ] JWT 인증 시 사용자 permission을 권한으로 부여
- [ ] 소유 관계 검사: 신청자는 자기 교육만, 강사는 자기 배정만
- 완료 기준: 권한 없는 요청 403, 남의 교육 조회 403 테스트

### 2. 기준 정보 (단순 등록·조회)
- [ ] cohort, program(배송·회수 일수), school(level, Address), school_level_setting
- [ ] application_source
- [ ] kit_type, program_kit_rule
- [ ] instructor_profile(type, school_id, region), instructor_program
- 완료 기준: Flyway 시드 데이터로 시연 가능

### 3. 신청 (EducationRequest)
- [ ] 운영자 신청 등록 (연락처 스냅샷, 신청자 계정 연결)
- [ ] 신청 목록: PriorityTier + PriorityReason 계산, 정렬
- [ ] 상태 전이: RECEIVED → REVIEWING → NEGOTIATING → CONFIRMED / REJECTED / WITHDRAWN
- [ ] 중복 표시: 운영자가 duplicate_of 지정 → DUPLICATE
- 완료 기준: 사전신청이 LMS 신청보다 항상 앞에 정렬되는 테스트

### 4. 교육 확정 (EducationClass, Session)
- [ ] 신청 → 교육 확정 (차시 여러 개, 날짜 + 시작 시간)
- [ ] 하루 최대 차시 검증 (school_level_setting)
- [ ] 배송지·교육 장소 상세 스냅샷
- [ ] Feasibility 조회: 교구 가용량, 배정 가능한 강사 수 (저장 안 함)
- [ ] 확정 시 ASSIGN_INSTRUCTOR Task 생성
- 완료 기준: 사례 3 인수 테스트 (신청 상태와 확정 상태 분리, 미검토 신청이 대시보드에 노출)

### 5. 교구 예약 (핵심 기술 포인트)
- [ ] 필요 수량 계산 (program_kit_rule)
- [ ] 점유 기간 계산 (첫 차시 − 배송일수 ~ 마지막 차시 + 회수일수)
- [ ] 예약: kit_type 행 `SELECT ... FOR UPDATE` → 겹치는 활성 예약 합계 → 보유량 초과 시 거절
- [ ] 상태: RESERVED → SHIPPED → RETURNED / RELEASED, 활성 예약 1건 제약
- [ ] 동시성 테스트: 남은 30개에 30개 예약 2건 동시 요청 → 1건만 성공
- 완료 기준: 동시성 테스트가 CI에서 안정적으로 통과

### 6. 강사 배정
- [ ] 배정 요청 (프로그램 가능 여부, 같은 날짜 중복 검사)
- [ ] 강사 본인만 수락·거절
- [ ] 수락 시 instructor_busy_date insert, 역할별 1명 제약
- [ ] StaffingCompositionPolicy: LEAD가 교사면 전원 교사 (수락 시 class 행 잠금)
- [ ] 교사 배정 수락 시 Task: RECEIVE_PRINCIPAL_APPROVAL, SEND_OFFICIAL_LETTER
- [ ] 배정 수락 시 ASSIGN_INSTRUCTOR Task 자동 종료
- 완료 기준: 허용 5개 조합 + 금지 조합 테스트, 같은 날짜 중복 배정 테스트

### 7. 변경 관리
- [ ] 운영자 직접 변경: 학생 수, 차시, 교사 직강 여부, 배송지·장소, 취소 (사유·채널 필수)
- [ ] change_event + change_detail 저장 (차시는 JSON 문자열)
- [ ] 신청자 ChangeRequest 생성 (승인 필요 항목만), 즉시 변경 항목(연락처, 확정 전 희망일)은 바로 반영
- [ ] 운영자 승인·반려, base_version 다르면 경고
- [ ] 취소 시 대기 중 ChangeRequest 자동 종료
- 완료 기준: 승인 전에는 교육 데이터가 바뀌지 않는 테스트

### 8. 영향 분석 + Task
- [ ] ImpactAnalyzer: 학생 수 변경 → 교구 수량 재검증 + UPDATE_SNACK
- [ ] 일정 변경 → 교구 재검증, CONFIRM_INSTRUCTOR_SCHEDULE, UPDATE_SNACK, NOTIFY_APPLICANT
- [ ] 취소 → 교구 RELEASED, 배정 CANCELLED, 열린 Task CANCELLED, busy_date 삭제
- [ ] 같은 종류 열린 Task는 새로 만들지 않고 갱신
- [ ] Task 처리: DONE / NOT_NEEDED + 사유 + 처리자
- [ ] 변경·이력·Task 저장을 한 트랜잭션으로
- 완료 기준: 사례 2 인수 테스트 (학생 수 변경 시 간식 Task 반드시 생성), 취소된 교육에 신규 Task 없음

### 9. 예외 대시보드
- [ ] Readiness 계산: 강사, 교구, 미처리 Task (READY / WARNING / BLOCKED)
- [ ] 문제 있는 교육 우선 목록: 강사 미배정, 교구 부족, 미처리 Task, 미검토 신청·ChangeRequest
- 완료 기준: 사례 1 인수 테스트 (출처와 무관하게 확정 교육이 목록에 나타남)

### 10. 셀프 조회
- [ ] 신청자: 내 교육 상태, 확정 일정, 학생 수, 강사 배정 상태, 교구 상태
- [ ] 강사: 내 배정 일정, 대기 중 요청

### 11. 마무리
- [ ] springdoc-openapi (Swagger UI)
- [ ] README: 문제 정의(장애 사례 3개), 아키텍처, ERD, 실행 방법, 기술적 고민(교구 동시성, 강사 중복 방지, 변경 영향 분석)
- [ ] 시연 시나리오 시드 데이터

---

## MVP2

- [ ] PostgreSQL 전환 (Flyway 스크립트 정리, Testcontainers 교체)
- [ ] DB 제약 고도화: 강사 시간대 overlap이 필요해지면 exclusion constraint
- [ ] 신청 import: `RequestImportPort` + CSV 어댑터, LMS 크롤링 어댑터, `(source, external_id)` 멱등
- [ ] 중복 신청 후보 자동 탐지 (판별 기준 확정 필요)
- [ ] 서류: DocumentType, DocumentRequirement(규칙 기반), DocumentSubmission(재제출 이력)
- [ ] 학교장 결재 Task를 DocumentRequirement로 전환
- [ ] 파일 저장 (비공개 스토리지 + 만료 URL), 개인정보 접근 로그, 보관·삭제 정책
- [ ] Readiness에 서류·배송 추가
- [ ] 신청자 자가 회원가입·사전 신청
- [ ] Transactional Outbox + 알림 (외부 부작용이 생길 때)
- [ ] 신청자 협조도 기록 (서류 지연, 연락 지연, 일정 변경 횟수) → 우선순위 반영
- [ ] 교구 파손·분실 처리
- [ ] AWS 배포

## 후순위
간식 시스템, 택배 API, 복잡한 알림, 강사료 정산, 결과보고 고도화, 챗봇
