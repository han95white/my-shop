# my-shop

B2C 쇼핑몰 백엔드. 학습 + 포트폴리오 목적의 1인 프로젝트.

## 기술 기준

- 저장소 루트 `my-shop/`, Spring 프로젝트는 `backend/`. 패키지 `com.back.shop`.
- Spring Boot 4.2.0, Java 21, Gradle(Groovy), 설정 파일은 YAML.
- 코드는 반드시 이 버전 기준으로 작성한다 (예전 Spring Boot 3 방식 금지).
- 의존성: Spring Web(webmvc), Spring Data JPA, MySQL Driver, Validation, Lombok, Flyway, Testcontainers.
- H2는 쓰지 않는다. DB가 필요 없는 로직은 순수 단위 테스트, DB가 필요한 테스트는 MySQL Testcontainers.
- 로컬 실행은 Docker + `./gradlew bootTestRun`.
- 공개 저장소다. 비밀번호·API 키는 커밋과 이슈에 절대 넣지 않는다 (환경변수로 분리).
- 프론트엔드는 나중에 같은 저장소의 `frontend/`에 추가한다. 지금은 백엔드 API 설계와 구현에만 집중한다.

## 범위

- 기본: 회원(JWT), 상품/카테고리, 장바구니, 주문, 결제(토스페이먼츠 테스트), 재고
- 특색 기능: 공동구매 (목표 인원 달성 시 확정, 마감까지 미달이면 자동 취소 + 일괄 환불)
- 제외: 배송 추적, 쿠폰, 리뷰. 범위 밖 기능은 제안하지 않는다.

## 설계 원칙 (반드시 지킨다)

1. 모든 프로파일에서 Flyway + `ddl-auto: validate`. 테스트는 MySQL Testcontainers.
   (이전 프로젝트: dev는 ddl-auto, prod만 Flyway라 마이그레이션 누락 발생)
2. Order-OrderItem을 하나의 애그리거트로. 상태 전이 규칙은 엔티티 안에 둔다.
   (이전: 검증 없는 상태 setter, 상태를 여러 서비스가 따로 변경)
3. check-then-act 금지. 재고 차감과 공동구매 참여 인원은 락 또는 조건부 UPDATE로 처리하고 동시성 테스트로 검증한다.
   (이전: 락 0건, 이중 처리 가능성)
4. 다른 도메인의 리포지토리 직접 주입 금지.
5. 목록 조회는 전부 페이징. 주문 시 가격은 스냅샷으로 저장한다.
6. 요청 검증은 Bean Validation을 쓴다.

## 작업 방식 (중요)

- 학습이 목적이므로 한 번에 많은 코드를 생성하지 않는다.
- 주문/재고/결제/공동구매 핵심 로직은 TDD로 한 단계씩, 왜 그렇게 짜는지와 다른 방법과의 비교를 설명하면서 진행한다.
- 엔티티/DTO 같은 반복 코드와 설정 파일은 바로 생성해도 된다.
- 선택지가 있는 결정은 장단점을 알려주고 사용자가 고르게 한다.
- 사용자가 모른다고 답하면 개념을 짧게 설명한 뒤 다시 묻는다.
- 중요한 결정은 ADR로 기록한다 (`docs/adr/`).

이번 프로젝트에서 새로 배우려는 것 — 해당 주제가 나오면 설명을 더 자세히 한다:
`@OneToMany` 애그리거트와 cascade / 락(`@Version`, 비관적 락, 조건부 UPDATE)과 동시성 테스트 / 외부 결제 연동과 멱등성 / 인덱스 설계와 EXPLAIN / 스케줄러(공동구매 마감 처리)

## 진행 순서

1. `/setup-matt-pocock-skills`
2. `/grill-with-docs`로 요구사항과 도메인 규칙 확정 — 재고, 결제 실패, 공동구매 마감, 환불 엣지 케이스를 집중적으로
3. `/to-spec` (ERD와 REST API 목록 포함)
4. `/to-tickets`
5. tdd로 티켓 하나씩 구현, 주기적으로 code-review

grill이 끝나기 전에는 기능 코드를 작성하지 않는다.

## Agent skills

### Issue tracker

이슈는 GitHub Issues(`han95white/my-shop`)에서 관리하며 `gh` CLI를 쓴다. See `docs/agents/issue-tracker.md`.

### Triage labels

기본 라벨 5개(`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`)를 그대로 쓴다. See `docs/agents/triage-labels.md`.

### Domain docs

single-context — 루트 `CONTEXT.md` + `docs/adr/`. See `docs/agents/domain.md`.
