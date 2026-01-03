# Java/Spring 개발자를 위한 Django 프로젝트 구조 이해

Java/Spring 환경에 익숙한 개발자가 Python/Django 프로젝트를 처음 접했을 때, 가장 혼란스러운 부분 중 하나는 프로젝트 구조와 요청 처리 흐름의 차이입니다. 이 문서는 Java/Spring의 일반적인 계층형 아키텍처(Controller, Service, Repository, DTO, Entity)와 Django의 구조를 비교하여 설명합니다.

## 1. Django 프로젝트의 기본 파일 구조 및 역할

-   `mysite/manage.py`: 프로젝트 관리를 위한 커맨드라인 유틸리티입니다. Spring Boot의 `mvnw` 또는 `gradlew` 스크립트와 유사한 역할을 수행합니다. (서버 실행, 데이터베이스 마이그레이션 등)

-   **`mysite/mysite/` (프로젝트 디렉토리)**
    -   `settings.py`: 프로젝트의 모든 설정을 담고 있는 파일입니다. Spring Boot의 `application.properties` 또는 `application.yml`에 해당하며, 데이터베이스, 설치된 앱, 미들웨어, 정적 파일 경로 등을 설정합니다.
    -   `urls.py`: 프로젝트의 최상위 URL 라우터입니다. 모든 HTTP 요청의 진입점 역할을 하며, 각 앱(app)의 `urls.py`로 요청을 분기합니다.

-   **`mysite/posts/` (앱 디렉토리)**
    -   `models.py`: 데이터베이스 테이블을 정의하는 파일입니다. Java/JPA의 **`@Entity`** 클래스와 완벽하게 일치하는 역할입니다.
    -   `views.py`: 요청을 받아 처리하고 응답을 반환하는 로직이 들어갑니다. Java/Spring의 **`@RestController`** 또는 **`@Controller`**에 해당합니다.
    -   `serializers.py`: `models.py`의 모델 인스턴스(객체)를 JSON 같은 형태로 변환하거나, 그 반대의 역할을 수행합니다. Java의 **DTO (Data Transfer Object)** 와 매우 유사한 개념입니다.
    -   `urls.py`: 이 앱(`posts`) 내부의 URL을 정의합니다. 프로젝트 `urls.py`에서 이곳으로 연결됩니다. Spring에서 컨트롤러 클래스나 메소드에 `@RequestMapping`을 선언하는 것과 같습니다.

## 2. Java(Spring)와 Django의 요청 처리 흐름 비교

두 프레임워크의 요청 처리 흐름을 비교하면 각 파일의 역할을 더 명확하게 이해할 수 있습니다.

### A. 일반적인 Java/Spring 웹 애플리케이션의 흐름

`Controller` -> `Service` -> `Repository` -> `Entity` -> `Database`

1.  **Client Request**: 클라이언트가 특정 엔드포인트(e.g., `/api/posts`)로 HTTP 요청을 보냅니다.
2.  **`Controller`**: `@RestController`가 요청을 받습니다. `@RequestBody`를 통해 JSON 데이터를 DTO로 변환하고, 기본적인 유효성 검사를 수행한 후 `Service` 계층의 메소드를 호출합니다.
3.  **`Service`**: 핵심 비즈니스 로직을 처리합니다. 트랜잭션 관리 등이 여기서 이루어지며, 데이터 처리를 위해 `Repository`를 호출합니다.
4.  **`Repository`**: `JpaRepository` 같은 인터페이스가 데이터베이스와의 상호작용을 담당합니다. `findById()`, `save()` 등의 메소드를 통해 `Entity` 객체를 조회하거나 저장합니다.
5.  **`Entity`**: `@Entity` 클래스는 데이터베이스 테이블과 매핑된 객체입니다.
6.  **Response**: `Service`는 결과를 `Controller`에 반환하고, `Controller`는 이 데이터를 다시 DTO로 변환하여 클라이언트에게 JSON 형태로 응답합니다.

### B. Django 웹 애플리케이션의 흐름

`project/urls.py` -> `app/urls.py` -> `views.py` -> (`serializers.py`) -> `models.py`(ORM) -> `Database`

1.  **Client Request**: 클라이언트가 특정 엔드포인트(e.g., `/posts/`)로 HTTP 요청을 보냅니다.
2.  **`project/urls.py`**: 프로젝트의 메인 URL 설정 파일이 요청을 받아, `posts/` 패턴과 일치하는 앱(`posts`)의 `urls.py`로 처리를 위임합니다.
3.  **`app/urls.py`**: `posts` 앱의 URL 설정 파일이 나머지 URL 패턴을 분석하여 실행할 `views.py` 안의 함수/클래스를 지정합니다.
4.  **`views.py` (Controller 역할)**: 지정된 뷰(View)가 요청을 처리합니다.
    -   **`serializers.py` (DTO 역할)**: 요청 본문의 JSON 데이터를 유효성 검사 후 파이썬 객체로 변환합니다.
    -   **`models.py` (Entity + Repository 역할)**: 뷰는 모델의 내장 ORM(`Post.objects.all()`, `Post.objects.create()` 등)을 직접 사용하여 데이터베이스와 상호작용합니다.
5.  **Response**: 뷰는 조회된 모델 객체(`QuerySet`)를 다시 **Serializer**를 통해 JSON 데이터로 변환하고, 클라이언트에게 `Response` 객체에 담아 반환합니다.

## 3. 핵심 차이점: Service와 Repository는 어디에 있는가?

-   **Repository 계층**: Django에서는 별도의 Repository 계층이 없습니다. **모델 클래스 자체가 ORM(`objects` 매니저)을 통해 Repository의 역할을 수행**합니다. `Post.objects.filter(...)`와 같은 호출이 바로 `postRepository.findBy...(...)`와 동일한 역할을 하는 것입니다.

-   **Service 계층**: Django는 명시적인 Service 계층을 강제하지 않습니다.
    -   **간단한 로직**: 대부분의 비즈니스 로직은 **`views.py`** 안에 바로 작성됩니다.
    -   **복잡한 로직**: 애플리케이션이 복잡해지면, 개발자들은 관례적으로 `views.py`가 비대해지는 것을 막기 위해 `services.py`라는 파일을 직접 만들어 비즈니스 로직을 분리하기도 합니다. 하지만 이는 프레임워크가 강제하는 구조가 아닌, 코드 유지보수를 위한 디자인 패턴에 가깝습니다.

## 결론

-   Django `views.py` = Spring **`Controller`**
-   Django `serializers.py` = Spring **`DTO`**
-   Django `models.py` = Spring **`Entity`** + **`Repository`**
-   Django에서 `Service` 계층은 명시적으로 분리되어 있지 않으며, 주로 `views.py`에 로직을 작성하거나 필요에 따라 수동으로 `services.py` 파일을 만들어 분리합니다.

이러한 차이점은 Django가 추구하는 "빠른 개발(Rapid Development)" 철학에서 비롯됩니다. 기본적인 CRUD 기능은 `views`, `models`, `serializers`만으로 매우 빠르게 구현할 수 있습니다.
