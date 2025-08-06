# Spring 기초
## 1. Spring MVC와 흐름에 대해 설명해주세요.
- **Model**  
  데이터 처리 및 비즈니스 로직을 담당  
  ex: `DTO`, `Service`, 등

- **View**  
  사용자가 보게 될 화면 또는 응답 결과  
  ex: `Thymeleaf`, `HTML` 등

- **Controller**  
  클라이언트의 요청을 받고, 적절한 로직을 수행한 뒤  
  Model 데이터를 View에 전달

### REST API 기반 Spring MVC 요청 처리 흐름

1. 클라이언트 요청
   - ex: `GET /api/users`
   - 클라이언트가 HTTP 요청을 보냄

2. DispatcherServlet이 요청 수신
   - Spring MVC의 프론트 컨트롤러 역할
   - 요청을 가로채고 처리 순서를 제어

3. HandlerMapping 조회
   - 해당 URI를 처리할 수 있는 `@Controller` 또는 `@RestController`를 탐색

4. HandlerAdapter 선택 및 실행
   - 선택된 Controller를 실행할 수 있는 HandlerAdapter를 찾고 실행

5. Controller 실행
   - 비즈니스 로직을 처리하고, DTO, List, Map 등의 객체를 반환

6. ViewResolver 생략
   - REST API 방식에서는 View 객체를 렌더링하지 않으므로 생략

7. HttpMessageConverter 동작
   - 반환된 객체를 `JSON`, `XML` 등으로 변환하여 HTTP 응답 본문에 담음

8. DispatcherServlet이 응답 반환
   - 최종적으로 변환된 JSON 응답을 클라이언트에 반환


## 2. Spring 애플리케이션을 계층 구조로 나눈다면 어떤 식으로 나누고, 각 계층의 역할은 무엇인가요?
| 계층 | 주요 역할 | 상세 설명 |
|------|-----------|------------|
| **Controller** | 요청 수신 및 응답 반환 | - 클라이언트 요청을 받고 파라미터를 검증한 뒤 Service에 전달<br>- Service의 결과를 가공해 클라이언트에 응답 |
| **Service** | 비즈니스 로직 처리 | - 도메인/비즈니스 로직 수행<br>- 데이터 가공 및 검증<br>- 필요한 경우 Repository를 호출 |
| **Repository** | DB 접근 및 데이터 처리 | - 실제 DB와 연결<br>- JPA 등을 통해 Entity 기반 쿼리 실행<br>- 데이터를 읽고/쓰기 |
| **Domain / Entity** | 도메인 모델, 데이터 구조 | - DB 테이블과 매핑<br>- 상태와 행위를 포함한 도메인 로직 보유 |


## 3. Spring에서 컨트롤러의 응답을 제어할 수 있는 방법에는 어떤 것들이 있나요?

### `@ResponseBody` / `@RestController`

- 메서드의 반환값을 HTTP 응답 본문에 직접 넣어 전달
- 객체 → JSON 등의 형태로 변환되어 응답
- 주로 간단한 REST API 응답에 사용됩니다.
- 단점: HTTP 상태 코드나 헤더를 세밀하게 제어할 수 없음

---

### `ResponseEntity<T>`

- 응답 **본문**, **HTTP 상태 코드**, **헤더**를 모두 명시적으로 제어할 수 있는 방식
- REST API 응답을 세밀하게 컨트롤하고자 할 때 사용

Q. 왜 응답 DTO에도 status를 사용했었는지?
```java
public class SuccessResponse<T> {
    private String status;
    private String message;
    private T data;
}
```
A. status가 상태를 표현하긴 하지만, 200 OK로 처리된 요청이라도 NO_RESULT, MODEL_SUCCESS 과 같이 좀 더 비즈니스적인 의미를 담은 상태를 표현하기 위함이였음

## 4. Spring에서 Bean을 관리하는 방법을 이야기해주세요.

### Bean 등록 방식
1. 클래스 단위 등록
   - `@Component`, `@Service`, `@Controller`, `@Repository` 등의 어노테이션 사용
   - Spring이 컴포넌트 스캔을 통해 자동으로 Bean으로 등록

2. 메서드 단위 등록
    - `@Configuration` + `@Bean`

### Bean Lifecycle
1. 객체 생성
    : 내부적으로 new 키워드로 객체 생성됨

2. 의존성 주입
    : 생성자, @Autowired, 필드 등으로 필요한 의존성 주입

3. (선택) 초기화 콜백
    : `@PostConstruct` 또는 `InitializingBean.afterPropertiesSet()`

4. 사용
    : 비즈니스 로직 수행

5. (선택) 소멸 콜백
    : `@PreDestroy` 또는 `DisposableBean.destroy()`

초기화/소멸 콜백은 필요한 경우에만 직접 정의

### 언제 초기화/소멸 콜백 필요?
- 컨트롤러, 서비스처럼 단순히 의존성 주입 받고, 로직 실행하는 경우에는 초기화 작업이 없어도 스프링이 자동으로 생성, 주입, 실행을 처리해줌

- 외부 API 연결, 리소스 초기화 등의 경우에는 초기화/소멸 콜백을 직접 작성

## 5. @Transactional을 메서드에 붙였는데도 롤백이 안 되면 어떤 이유가 있을까요?

### 1. Checked Exception 발생 시
- Java의 Checked Exception(예: `IOException`, `FileNotFoundException`)은  
  **컴파일 시점에 예외 처리를 강제하는 예외**
- `@Transactional`은 기본적으로 `RuntimeException`이나 `Error`가 발생했을 때만 **롤백**
- -> `Checked Exception`이 발생해도 트랜잭션이 롤백X
- 만약 이런 Checked Exception이 발생해도 롤백이 되게 하기 위해서는 `rollbackFor = Exception.class` 옵션을 붙여주면 됨

### 2. 같은 클래스 내부에서 @Transactional 메서드를 호출한 경우
- `@Transactional`은 프록시 기반 AOP로 동작
- 즉, Spring이 생성한 프록시 객체가 메서드를 감싸야 트랜잭션 로직이 적용
- But, 같은 클래스 내부에서 메서드를 호출하면 프록시 객체를 거치지 않고, 실제 인스턴스의 메서드가 호출 -> 트랜잭션 적용X
- 이런 경우 보통 트랜잭션 메서드를 **다른 서비스 클래스**로 분리하거나 복잡한 로직의 경우 **Facade 패턴**을 사용해서 해결

### 3. 예외를 try-catch로 잡고 무시하는 경우
- 트랜잭션은 예외가 밖으로 전파되어야만 롤백함
- `try-catch`로 예외를 잡고 아무 처리X -> 스프링은 예외 발생 여부를 알 수 X -> 롤백X
- 이를 해결하기 위해서는 `catch` 블록에서 예외를 다시 throw 하거나 `setRollbackOnly()`를 설정
```java
catch (Exception e) {
    TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
}
```
