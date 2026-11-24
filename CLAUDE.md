# Massac: AI 작업 규칙

정부지원 교육사업 운영 관리 시스템. 도메인 상세는 `docs/domain.md`를 먼저 읽는다.
태그 의미: [확정] 업무 사실 또는 결정됨 / [가설] 설계 제안, 바뀔 수 있음 / [미정] 정보 없음, 가정 금지.

## 답변 원칙
1. 업무 사실([확정])이 기존 코드·ERD보다 우선한다. 규칙을 지키려고 설계를 바꾸는 건 되고, 설계를 지키려고 규칙을 왜곡하는 건 안 된다.
2. [미정] 항목은 임의로 채우지 않는다. 필요하면 질문부터 한다.
3. 설계를 바꾸자고 할 때는 "현재 구조 / 개선 구조 / 바꾸는 이유 / 예외 시나리오"를 함께 쓴다.
4. 사실과 제안을 섞지 않는다. 제안에는 [제안]이라고 표시한다.
5. 서류 이름 등 법적 명칭은 확인되지 않았으면 확정하지 않는다.
6. Java 17, Spring Boot, Spring Data JPA에서 바로 구현 가능한 수준으로 제안한다.

## 답변 형식 (설계 질문일 때)
1. 결론 한두 문장
2. 관련 [확정] 규칙
3. 현재 vs 개선 비교표
4. 예외 시나리오
5. 결정이 필요한 질문

## 핵심 불변식 [확정]
1. 신청자는 확정된 교육의 주요 정보를 직접 수정할 수 없다.
2. 신청자의 중요 변경은 ChangeRequest 승인 후에만 실제 데이터에 반영한다.
3. 운영자가 주요 교육정보를 직접 변경하면 변경 사유가 필요하다.
4. 주요 변경은 추적 가능한 이력을 남긴다.
5. 같은 기간의 유효한 교구 예약 총량은 보유량을 넘을 수 없다.
6. 같은 강사는 같은 날짜에 여러 교육에 확정 배정될 수 없다.
7. 강사는 자신에게 요청된 Assignment만 수락·거절할 수 있다.
8. 필요한 서류는 사용자 역할과 교육 조건에 따라 결정된다.
9. 취소된 교육에는 정상 진행용 신규 업무를 만들지 않는다.
10. 교육 변경으로 생긴 필수 후속 업무는 장애로 유실되면 안 된다.

## 금지
- 모든 기능을 CRUD로만 표현하기
- Priority와 Feasibility를 하나로 합치기
- 신청(EducationRequest)과 확정 교육(EducationClass)을 합치기
- 서류 요구사항과 제출 파일을 합치기
- "운영자가 알아서 처리"를 전제로 설계하기 (기억 의존 제거가 목적)
- 모든 것을 자동화하기 (외부 협의·판단은 사람이 한다)
- 대규모 트래픽 시스템처럼 과장하기, 불필요한 Kafka·MSA 도입

## 코드 규칙
- 패키지: 도메인별 `controller / service / repository / dto / entity / exception`
- 예외: 도메인별 `ErrorCode` enum + `CustomException`, 응답은 `ErrorResponse`
- 삭제: `Status.DELETED` 소프트 삭제
- 트랜잭션: Spring `@Transactional` 사용
- 새 기능은 테스트를 먼저 쓰고, `./gradlew test` 전체 통과를 완료 기준으로 한다. 테스트를 고치거나 건너뛰어서 통과시키지 않는다.
- 커밋 메시지: `feat : 한글 설명`
- 비밀값(DB 비밀번호, JWT 키)은 환경변수로만 다룬다.

## 명령어
- 테스트: `./gradlew test`
- 단일 테스트: `./gradlew test --tests "com.mink.massac.MassacApplicationTests"`
- 로컬 DB: `docker compose up -d` (MySQL 8.4, Redis 7.4)
- 실행: `./gradlew bootRun`
- 스키마 변경은 `src/main/resources/db/migration`에 Flyway 스크립트를 추가한다 (`ddl-auto=validate`)
- 통합 테스트는 `support/TestcontainersConfiguration`을 `@Import`한다 (Docker 필요)
