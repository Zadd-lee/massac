# Massac

[![CI](https://github.com/Zadd-lee/massac/actions/workflows/ci.yml/badge.svg)](https://github.com/Zadd-lee/massac/actions/workflows/ci.yml)

정부지원 교육사업 운영 관리 시스템.

여러 시스템(LMS, 구글 시트, 카카오톡, 전화)에 흩어져 있던 교육의 신청·일정·강사·교구 데이터를 하나의 교육 단위로 모으고, 일정·학생 수 변경이 연관 업무에 주는 영향을 추적해 운영 누락을 막습니다.

> 작성 중입니다. 현재 MVP1 0단계(프로젝트 셋업)까지 진행했습니다. 진행 상황은 [docs/backlog.md](docs/backlog.md)를 보세요.

## 해결하려는 문제

실제 운영에서 겪은 장애 사례가 출발점입니다.

1. **사전신청 교육 누락**: LMS 기준으로 만든 운영 시트에서 사전신청 교육이 빠져, 교육 7일 전에야 발견
2. **학생 수 변경 전파 실패**: 운영 시트만 고치고 간식 업체 시트는 그대로, 당일 간식 부족
3. **신청과 확정 혼동**: 신청자는 LMS 신청 완료를 확정으로 이해, 운영팀은 미확인

자세한 도메인 설명: [docs/domain.md](docs/domain.md) / ERD: [docs/erd.md](docs/erd.md)

## 기술 스택

Java 17, Spring Boot 4.1, Spring Data JPA, Spring Security, MySQL 8, Redis 7, Flyway, JUnit 5, Testcontainers, GitHub Actions

## 실행 방법

필요한 것: JDK 17, Docker

```bash
# 1. MySQL, Redis 실행
docker compose up -d

# 2. 테스트 (Testcontainers가 테스트용 DB를 따로 띄움)
./gradlew test

# 3. 애플리케이션 실행
./gradlew bootRun
```
