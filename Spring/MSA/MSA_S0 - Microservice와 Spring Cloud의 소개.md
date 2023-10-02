# S0 : Microservice와 Spring Cloud의 소개

Written By [Ajeong Joo](https://github.com/ajung7038)

## ✨ 소프트웨어 아키텍처

### Antifiagile의 네 가지 특징

- Auto scalling
  -> 자동 확장성을 가짐 (자동 인스턴스 증가)
  -> 사용량이 많아질 때는 서버의 운영 개수를 늘리고, 나머지는 줄이는 방식
- Microservices
  -> Cloud Native Application의 핵심
  -> 세분화된 서비스
- Chaos engineering
  -> 운영 중인 소프트웨어 시스템의 실행 방법 or 규칙
- Continuous deployments (CI/CD)
  -> 파이프라인으로 연결시켜 둠.
  -> 통합적인 관리와 운영을 위해 사용

## ✨ Cloud Native Architecture

### 확장 가능한 아키텍처

- 수평적 확장 : 더 많은 사용자의 요청 처리 가능
- scale up : 하드웨어의 사용량을 높임
- scale out : 인스턴스를 여러 대 배치 (요청 처리 多)
- 필요한 만큼 빌려 비용을 최소화시킴

### 탄력적 아키텍처

- 서비스 생성 - 통합 - 배포, 비즈니스 환경 변화에 대응 시간 단축
- 분리된 서비스 간 통신을 위해 종속성을 최소화시켜야하고, 상태를 갖지 않아야 함

### 장애 격리 (Fault isolation)

- 하나의 Microservice Application에 생기는 문제나 오류는 다른 서비스에 영향을 주지 않는다.

## ✨ Cloud Native Application

### 정의

- Cloud Native Architecture에 의해 설계되고 구현된 애플리케이션

### 구현

- Microservices로 개발
- CI/CD : 시스템에 의해 자동 통합, 빌드, 테스트, 배포
- DevOps : 시스템이 종료될 때까지 무한 반복 -> 최선의 결과물 생성
- Containers

### 특징

#### 1. CI/CD와 배포

> `CI`

- 지속적인 통합, CI (Continuous Integration)
- 두 가지 의미
  1.  결과물을 통합하기 위한 형상 관리
  2.  통합된 코드 관리 및 테스트 자체
- 깃과 같은 시스템과 연동해 사용

> `CD`

- 두 가지 의미
  1. Continuous Delivery
  2. Continuous Deployment
- 실행 환경에 어떻게 배포하는지에 따라 달라짐
  -> 수동 반영 시 1
  -> 자동 반영 시 2
- pipe line

> `배포`

- 카나리 배포와 블루그린 배포
- 5프로만 옮기거나, 전체를 옮기거나

#### 2. DevOps

- Development + Operations
- 목적 : 고객의 요구사항을 빠르게 반영하고 만족도 높은 결과물 제시
- 전체 개발 일정이 완료될 때까지 지속적으로 진행하는 것

#### 3. Container 가상화

- Cloud Native Architecture의 핵심
- 적은 리소스를 사용하여 가상화 구축

#### 컨테이너 가상화 기술의 발전

- Traditional Deployment -> Virtualized Deployment -> Container Deployment

- cf) Virtualized Deployment : host system을 쪼개서 사용하는 것 -> 독립적인 운영체제를 가짐

## ✨ 12 Factors

- Cloud Native Application 개발 및 운영 시 고려해야할 항목 12가지
- 이를 바탕으로 설계, 개발 및 운영

### 항목

1. Base Code
   -> 코드를 한 곳에서 배포하는 것이 주 목적 (형상 관리)
   -> 가장 중요한 항목 (통일적 관리가 필요하기 때문)

2. Dependency Isoluation
   -> 자체 종속성을 가지고 전체 시스템에 영향을 끼치지 않아야 함

3. Configurations
   -> 코드 외부에서 구성 관리 도구를 사용해 Microservice에 필요한 작업 제어

4. Linkable Backing Services
   -> 보조 서비스 이용 : 추가 기능 지원 (Code dependency X)

5. Stages Of Creation
   -> Build, Release, Run 분리
   -> 고유한 아이디로 태그, 롤백 기능 지원, CI/CD 시스템 (자동화) 구축

6. Statelees Processes
   -> 자체 프로세스에서 운영할 수 있어야 한다 (독립성과 일치)

7. Port Binding
   -> 자체 포트에서 노출되는 인터페이스 및 자체 포함 기능이 있어야 한다 (격리 O)

8. Concurrency
   -> 동일한 프로세스 복사 -> 확장

9. Disposability
   -> 서비스 인스턴스 자체의 기능(등록, 삭제 등)을 가능하게 해야한다

10. Development & Production Parity
    -> 수명 주기 전반에 걸쳐 환경 자체를 중복되지 않고, 종속적이지 않게 서비스를 유지해야 한다

11. Logs
    -> Microserveice에 의해 생성된 로그를 이벤트 스트림으로 처리해야한다
    -> Application이 실행되지 않는 상태라고 하더라도 로그가 작동되어야 한다
    -> ex) Azure, 데이터 마이닝

cf) `Azure` : 마이크로 사의 클라우드 컴퓨팅 플랫폼 (AWS 같은 거)

12. Admin Processes For Eventual Processes
    -> 리소스 사용을 파악할 수 있는 적절한 도구가 있어야 한다
    -> 데이터 정리 및 데이터 분석 도구

- 최근에는 +3이 추가되는 추세
  -> API first, Telemetry, Authentication and autorization 추가

## ✨ Monolithic vs MSA

- Application 개발 방법 중 일부
- `하나의 서비스를 구성하고 있는 크기` 가 가장 큰 차이점

### Monolith

- 필요한 모든 요소를 하나의 소프트웨어 안에서 처리
- 서로 의존성을 가진 채 패키징
- 하나의 커다란 건축물로 이해하면 편하다
- 데이터베이스 자체도 한 곳에 집중된다 (하나의 공통적인 데이터베이스 사용)
- 문제점 : 시스템의 일부만 수정한다고 하더라도 전체에 영향이 끼친다

### Front & Back

- Monolith 방식과 Microservice Architecture 방식의 중간 방식
- 프론트와 백을 분리
  -> 모바일 애플리케이션에서 많이 사용되는 방식
  -> HTTP, 프로토콜 등으로 통신
- 최적화되어 있는 환경을 독립적으로 유지

### Microservices

- "함께 작동하는 작은 규모의 서비스들"
- 각각의 구성 요소 및 서비스 분리
- 유지 보수나 변경 사항을 적용하는 데 용이
- 독립적으로 배포 가능
- 업데이트 전체가 다운타임 되는 현상 방지
- 비즈니스 기능을 중심으로 구축되어야 한다

#### 프로젝트 서비스, 레포 서비스 등 백엔드 부분을 독립적인 서비스로 분리

## ✨ Microservice Architecture

- 모든 것이 다 Microservice는 아니어도 된다
- 서비스 간 종속성 최소화, 응집력을 높일 수 있게 하는 것

## ✨ SOA vs MSA

### 개념

- SOA : 서비스 지향 아키텍처
  -> 대규모 컴퓨터 시스템을 구축할 때 개념
  -> 업무상의 일 처리에 해당하는 소프트웨어 기능을 서비스로 판단하여 그 서비스를 네트워크상에 연동하여 시스템 전체를 구축해 나가는 방법론

### 차이점

- 서비스의 공유 지향점은 같으나, 서비스 공유 크기에 차이가 있다.
- SOA
  -> 재사용을 통한 비용 절감
  -> 서비스 공유 최대화
  -> 기술 방식 : 공통 서비스 형식으로 서비스 제공

- MSA
  -> 서비스 간의 결합도를 낮추어 변화에 능동적으로 대응
  -> 서비스 공유 최소화
  -> REST API 사용

### REST API 설계 시 고려해야할 사항

- 소비자 입장에서의 설계 방식 채택
- HTTP의 장점을 살려야 한다
  ->헤더 값, response 타입 등
- Request methods
  -> Lv2 이상의 모델 특징을 살려야 한다
- Response Status
  -> 적절한 HTTP 상태 코드가 전달되어야 한다
  -> 오류 이유 또한 반환시켜주는 것도 필요하다

> ex) /users/10
> -> 10번 회원이 없다! (데이터베이스 저장 X)
> -> 상태 코드로 몇 번을 전달해야할까?
> -> 500? (서버는 정상)
> -> 200? (클라이언트는 값을 못 가짐) -> 상황에 따라 200번 코드도 가능하지만 404가 더 적절하다
> => 404!

- URI
- 단수 형태의 URI 값보다 복수 형태의 URI를 지향 해야 한다
- 명사 형태로 표시해서 사용자에게 직관적으로 전달 해야 한다

cf) 헷갈려서 정리하는 `인스턴스` 의 개념

## ✨ Microservice Architecture Structure

![](https://velog.velcdn.com/images/ajeong7038/post/31dd727e-d99f-433d-aea8-9f8aaabcb132/image.png)

### Service Mesh

- Microservice Architecture를 적용한 시스템의 내부 통신
- 하나의 제품이나 서비스가 아닌 추상적 개념
- cf) Routing, Proxy...

## ✨ Spring Cloud

### Spring Cloud Config Server

- 환경설정
- 서비스들 -> 클라우드 서비스 -> git (토큰 기본 정보 등)

### Load Balancing

- 부하분산 또는 로드 밸런싱은 컴퓨터 네트워크 기술의 일종으로 둘 혹은 셋이상의 중앙처리장치 혹은 저장장치와 같은 컴퓨터 자원들에게 작업을 나누는 것을 의미한다.
- 가용성 및 응답시간을 최적화 시킬 수 있다.

## ✨ 참고 자료

- [Load Balancing]
- [SOA]

[Load Balancing]: https://ko.wikipedia.org/wiki/%EB%B6%80%ED%95%98%EB%B6%84%EC%82%B0
[SOA]: https://ko.wikipedia.org/wiki/%EC%84%9C%EB%B9%84%EC%8A%A4_%EC%A7%80%ED%96%A5_%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98
