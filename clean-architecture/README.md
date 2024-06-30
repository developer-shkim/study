> 책 <만들면서 배우는 클린 아키텍처> 의 내용과 느낀 점들을 간단히 정리했습니다.
>
> 출처: 책 <만들면서 배우는 클린 아키텍처> - 톰 홈버그 지음, 박소은 옮김, 조영호 감수, 위키북스 출판

# 클린 아키텍처 구조

![](https://reflectoring.io/images/posts/spring-hexagonal/hexagonal-architecture_hu6764515d7030d45af6f7f498c79e292b_50897_956x0_resize_box_3.png)

```
.
├── README.md
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    ├── main
    │   ├── java
    │   │   └── io
    │   │       └── reflectoring
    │   │           └── buckpal
    │   │               ├── BuckPalApplication.java
    │   │               ├── BuckPalConfiguration.java
    │   │               ├── BuckPalConfigurationProperties.java
    │   │               ├── account
    │   │               │   ├── adapter
    │   │               │   │   ├── in
    │   │               │   │   │   └── web
    │   │               │   │   │       └── SendMoneyController.java
    │   │               │   │   └── out
    │   │               │   │       └── persistence
    │   │               │   │           ├── AccountJpaEntity.java
    │   │               │   │           ├── AccountMapper.java
    │   │               │   │           ├── AccountPersistenceAdapter.java
    │   │               │   │           ├── ActivityJpaEntity.java
    │   │               │   │           ├── ActivityRepository.java
    │   │               │   │           └── SpringDataAccountRepository.java
    │   │               │   ├── application
    │   │               │   │   ├── port
    │   │               │   │   │   ├── in
    │   │               │   │   │   │   ├── GetAccountBalanceQuery.java
    │   │               │   │   │   │   ├── SendMoneyCommand.java
    │   │               │   │   │   │   └── SendMoneyUseCase.java
    │   │               │   │   │   └── out
    │   │               │   │   │       ├── AccountLock.java
    │   │               │   │   │       ├── LoadAccountPort.java
    │   │               │   │   │       └── UpdateAccountStatePort.java
    │   │               │   │   └── service
    │   │               │   │       ├── GetAccountBalanceService.java
    │   │               │   │       ├── MoneyTransferProperties.java
    │   │               │   │       ├── NoOpAccountLock.java
    │   │               │   │       ├── SendMoneyService.java
    │   │               │   │       └── ThresholdExceededException.java
    │   │               │   └── domain
    │   │               │       ├── Account.java
    │   │               │       ├── Activity.java
    │   │               │       ├── ActivityWindow.java
    │   │               │       └── Money.java
    │   │               └── common
    │   │                   ├── PersistenceAdapter.java
    │   │                   ├── SelfValidating.java
    │   │                   ├── UseCase.java
    │   │                   └── WebAdapter.java
    │   └── resources
    │       └── application.yml
    └── test
        ├── java
        │   └── io
        │       └── reflectoring
        │           └── buckpal
        │               ├── BuckPalApplicationTests.java
        │               ├── DependencyRuleTests.java
        │               ├── SendMoneySystemTest.java
        │               ├── account
        │               │   ├── adapter
        │               │   │   ├── in
        │               │   │   │   └── web
        │               │   │   │       └── SendMoneyControllerTest.java
        │               │   │   └── out
        │               │   │       └── persistence
        │               │   │           └── AccountPersistenceAdapterTest.java
        │               │   ├── application
        │               │   │   └── service
        │               │   │       └── SendMoneyServiceTest.java
        │               │   └── domain
        │               │       ├── AccountTest.java
        │               │       └── ActivityWindowTest.java
        │               ├── archunit
        │               │   ├── Adapters.java
        │               │   ├── ApplicationLayer.java
        │               │   ├── ArchitectureElement.java
        │               │   └── HexagonalArchitecture.java
        │               └── common
        │                   ├── AccountTestData.java
        │                   └── ActivityTestData.java
        └── resources
            └── io
                └── reflectoring
                    └── buckpal
                        ├── SendMoneySystemTest.sql
                        └── account
                            └── adapter
                                └── out
                                    └── persistence
                                        └── AccountPersistenceAdapterTest.sql

47 directories, 52 files
```

# 요약

### 01. 계층형 아키텍처의 문제는 무엇일까?

- 계층형 아키텍처에서 적용되는 전통적인 규칙은 같은 계층 또는 하위 계층만 접근 가능
- 상위 계층에 접근하려면 설계와 맞지 않게 컴포넌트가 아래로 내려질수도 있음
- 웹 -> 도메인 -> 영속성 구조로 하면, 도메인 로직이 아닌 데이터베이스 중심으로 개발될 수 있음
- 계층의 너비 규칙이 강제되지 않아서, 특정 유스케이스를 찾기 어려움
  - userService 대신 registerUserService 와 같이 특정 유스케이스별로 생성 권장
  - 같은 service 동시에 편집하면, 병합 시에 충돌 될 수 있어서 이전 코드로 되돌려야 하는 잠재적 문제 있음

### 02. 의존성 역전하기

- 단일 책임 원칙: 컴포넌트를 변경하는 이유는 오직 하나뿐이어야 한다.
- 전이 의존성, 순환 의존성에 주의해야 함
- 웹 -> 도메인 <- 영속성 구조로 설계하고, 도메인 계층에서 제공하는 인터페이스를 각각 구현하도록 하여 의존성 역전
- 입력 포트에 대한 주도하는(driving) 어댑터, 출력 포트에 대한 주도되는(driven) 어댑터
- 헥사고널 아키텍처에서 의미상 그런 것이지 꼭 육각형이라는 것에 의미가 있는 것은 아님
- 어댑터 -> 애플리케이션 계층 -> 도메인 엔티티 순으로 의존함

### 03. 코드 구성하기

- 서로 연관되는 기능끼리 묶고
- 애플리케이션이 어떤 유스케이스를 제공하는지 알 수 있고
- 패키지 구조에 아키텍처가 드러나도록
- adapter 패키지 package-private
  - application 패키지 내의 포트 인터페이스 통하지 않고는 바깥에서 호출되지 않으므로
- domain 패키지의 도메인 클래스는 public
- service 는 어댑터에 의해 숨겨지므로 public 일 필요 없음

### 04. 유스케이스 구현하기

- 입력 유효성 검증
  - 구문상의 유효성 검증, 모델 접근하지 않고도 알 수 있음
  - 애플리케이션 코어 밖으로부터 유효하지 않은 값을 받으면 모델 상태 해칠 수 있으므로 애플리케이션 계층에서 검증함
  - 입력 모델(예: SendMoneyCommand) 생성자에서 검증하는 방법도 있음
- 비즈니스 규칙 검증
  - 도메인 엔티티 안에 넣으면 비즈니스 로직 옆에 규칙이 바로 위치해서 인지하기 쉬움
  - 유스케이스에서도 가능
- 유스케이스끼리 출력 모델을 공유하거나 도메인 엔티티를 출력 모델로 하는 것은 변경 이유가 늘어나므로 권장 하지 않음
- 읽기 전용 유스케이스는 query 로, 쓰기 전용은 command 로 해서 간단한 데이터 조회는 유스케이스와 구분 권장

### 05. 웹 어댑터 구현하기

- 어댑터와 유스케이스 사이에 포트와 같은 계층을 넣어야 할까?
  - 애플리케이션 코어가 외부 세계와 통신할 수 있는 곳에 대한 명세가 포트임
  - 외부와 어떤 통신이 일어나고 있는지 알 수 있고, 레거시 다루는 엔지니어에게 정보 제공
- 웹 어댑터 역할
  - 1. http 요청을 자바 객체로 매핑
  - 2. 권한 검사
  - 3. 입력 유효성 검사
  - 4. 입력을 유스케이스 입력 모델로 매핑
  - 5. 유스케이스 호출
  - 6. 유스케이스 출력을 http로 매핑
  - 7. http 응답 반환

### 06. 영속성 어댑터 구현하기

- 영속성 어댑터 역할
  - 1. 입력을 받음
  - 2. 입력을 데이터베이스 포맷으로 매핑
  - 3. 입력을 데이터베이스로 보냄
  - 4. 데이터베이스 출력을 애플리케이션 포맷으로 매핑
  - 5. 출력을 반환
- 포트 인터페이스 나누기
  - 넓은 포트 인터페이스에 의존성을 갖고 코드에 불필요한 의존성 생김
  - 유스케이스별로 인터페이스 나누고, 어댑터가 여러 인터페이스를 구현하도록 권장
- 애그리거트당 하나의 영속성 어댑터
  - 여러 바운디드 컨텍스트의 영속성 요구사항 분리하기 위한 좋은 토대가 됨
- 영속성 측면과의 타협 없이 풍부한 도메인 모델 생성하려면 도메인 모델과 영속성 모델 매핑
- 트랜잭션은 유스케이스에서 하거나 위빙
  - 영속성 어댑터는 어떤 연산이 같은 유스케이스로 묶이는지 알 수 없음
  - 또는 관점 지향 프로그래밍으로 트랜잭션 경계를 코드에 위빙할 수도 있음

### 07. 아키텍처 요소 테스트하기

![](https://semaphoreci.com/wp-content/uploads/2022/03/pyramid-cost.webp)

- 종류
  - 시스템 테스트: 애플리케이션 구성하는 모든 객체 네트워크 가동 -> 특정 유스케이스가 전계층에서 잘 작동하는지 검증
  - 통합 테스트: 여러 유닛 인스턴스화, 시작점 인터페이스에 데이터 전송
  - 단위 테스트: 하나의 클래스 인스턴스화, 나머지 의존 클래스는 mocking
- 모든 동작을 테스트하려고 하면, 클래스 변경 시마다 테스트도 변경해야 하므로 어떤 상호작용을 검증하고 싶은지 신중하게 결정해야 함
- 영속성 어댑터 테스트 시에는 인메모리 DB 보다는 실제에 가깝게 테스트 권장
  - testcontainers 같은 라이브러리 사용하면 필요한 DB 를 도커 컨테이너에 띄울 수 있음
- 테스트 전략
  - 단위 테스트: 도메인 엔티티, 유스케이스
  - 통합 테스트: 어댑터
  - 시스템 테스트: 사용자가 취할 수 있는 중요 애플리케이션 경로

### 08. 경계 간 매핑하기

- 간단한 CRUD 케이스는 매핑하지 않고 포트 인터페이스가 도메인 모델을 입출력 모델로 사용하고, 추후에 다른 전략으로 변경
- 매핑 전략
  - 양방향(Two-Way): 각 계층이 전용 모델을 가짐 (예: 웹 계층 - 웹 모델, 애플리케이션 계층 - 도메인 모델, 영속성 계층 - 영속성 모델)
  - 완전(Full): 양방향 매핑 + 입출력 모델을 가짐 (예: SendMoneyCommand, UpdateAccountStateCommand)
  - 단방향(One-Way): 모든 계층 모델이 같은 인터페이스 구현, 도메인 모델 getter 제공해서 다른 계층에서 매핑 없이 사용
- 가이드라인 (하지만, 상황에 맞게 그때그때 다르게 사용)
  - 변경 유스케이스
    - 웹, 애플리케이션 사이에는 양방향 매핑을 사용해서 유스케이스별 유효성 검증을 명확하게 특정 유스케이스에 필요없는 필드는 없이 받음
    - 애플리케이션, 영속성 사이에는 매핑을 하지 않거나 양방향으로만 매핑하여 매핑 오버헤드를 줄이고 빠른 코딩
  - 쿼리 유스케이스는 매핑하지 않는 전략 권장

### 09. 애플리케이션 조립하기

- 유스케이스는 인터페이스만 알아야 하고, 런타임에 구현을 제공받아야 함
- 아키텍처에 중립적이고 인스턴스 생성을 위해 모든 클래스에 대한 의존성을 가진 설정 컴포넌트가 있어야 함
- 설정 컴포넌트 역할
  - 웹 어댑터 인스턴스 생성
  - http 요청이 실제로 웹 어댑터로 전달되도록 보장
  - 유스케이스 인스턴스 생성
  - 웹 어댑터에 유스케이스 인스턴스 제공
  - 영속성 어댑터 인스턴스 생성
  - 유스케이스에 영속성 어댑터 인스턴스 제공
  - 영속성 어댑터가 실제 데이터베이스에 접근할 수 있도록 보장
- 스프링의 클래스패스 스캐닝, 자바 컨피그 방식이 있음

### 10. 아키텍처 경계 강제하기

- package-private 제한자
  - 패키지 내에 있는 클래스들은 서로 접근 가능하지만, 바깥에서는 접근할 수 없음
  - 모듈의 진입점으로 활용될 클래스들만 골라서 public 으로 만들면 됨
  - adpater, service 에 설정
- 컴파일 후 체크 단계 도입
  - 코드가 컴파일 된 후 런타임에 의존성 규칙 위반 여부 확인
  - ArchUnit 같은 도구로 적용 가능
  - 아키텍처 내 관련된 모든 패키지 명시할 수 있는 DSL 만들 수 있음 (예: src/test/java/io/reflectoring/buckpal/archunit/HexagonalArchitecture.java)
- 빌드 아티팩트
  - 빌드 도구에서 의존성을 해결할 때, 사용 불가능한 것이 있으면 빌드가 실패함
  - 이를 활용해서 각 계층에 대해 분리된 빌드 모듈을 생성
  - 패키지로 구분한 것과 비교해서 아래와 같은 장점 있음
    - 순환 의존성이 없음을 확신할 수 있음
    - 다른 모듈 고려하지 않고 격리한채로 변경 가능(다른 모듈 컴파일 에러 있더라도 작업 가능)

### 11. 의식적으로 지름길 사용하기

- 인커밍 포트 건너뛰기
  - 아웃고잉 포트는 의존성 역전을 위해 필수지만, 인커핑 포트는 필수는 아님
  - 인커밍 포트 없이 인커밍 어댑터가 애플리케이션 서비스에 직접 접근한다면 특정 유스케이스 구현 위해 어떤 서비스 메서드 호출해야 할지 알아내기 위해 내부 동작을 잘 알아야 함
- 인커핑 포트 유지한다면
  - 유스케이스 구현할 때, 진입점을 식별하기 쉬움
  - 인커밍 어댑터가 인커밍 포트만 호출할 수 있도록 강제할 수 있어서 의도하지 않은 서비스 메서드 실수로 호출하지 않게 함

# 출처

- https://reflectoring.io/spring-hexagonal/
- https://semaphoreci.com/blog/testing-pyramid
