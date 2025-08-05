
## 1-1. Spring 컨테이너에 대해 설명해주세요.

Spring 컨테이너는 **Spring Framework의 핵심**으로, **객체의 생성부터 소멸까지 전체 생명주기를 관리하는 실행 환경**입니다.

### **컨테이너의 핵심 역할:**

**첫 번째, IoC(제어의 역전) 구현**입니다. 기존에는 개발자가 직접 `new` 키워드로 객체를 생성하고 의존관계를 설정했다면, Spring 컨테이너가 이 모든 제어권을 가져와 대신 처리합니다.

**두 번째, DI(의존성 주입) 수행**입니다. 객체 간의 의존관계를 컨테이너가 자동으로 연결해주어, 각 객체는 자신의 비즈니스 로직에만 집중할 수 있습니다.

### **컨테이너의 동작 과정:**

컨테이너는 시작 시 **설정 정보(@Configuration, @ComponentScan 등)를 읽어** Bean의 메타데이터를 수집합니다. 이를 바탕으로 **BeanDefinition을 생성**하고, **실제 객체를 인스턴스화**한 후 **의존성 주입과 초기화**를 수행하여 Bean을 사용 가능한 상태로 만듭니다.

### **주요 장점:**

**느슨한 결합**으로 코드 유연성이 향상되고, **Singleton 자동 관리**로 메모리 효율성을 제공하며, **AOP, 트랜잭션 등의 횡단 관심사**를 선언적으로 처리할 수 있습니다. 또한 **테스트 시 Mock 객체 주입**이 용이하여 테스트 작성이 편리합니다.

결론적으로 Spring 컨테이너는 **객체 관리의 모든 복잡성을 대신 처리**해주어, 개발자가 비즈니스 로직에만 집중할 수 있게 해주는 핵심 인프라입니다.

**꼬리질문:** Spring 컨테이너는 언제 생성되고 언제 소멸되는지 설명해주실 수 있나요?

→ 네, Spring Container는 애플리케이션이 실행될 때 가장 먼저 생성됩니다.
구체적으로 말씀드리면, main 메소드에서 SpringApplication.run()을 호출하는 그 순간부터 Container 생성이 시작됩니다.

초기화 과정을 단계별로 설명드리면, 먼저 클래스패스를 스캔해서 @Component, @Service, @Bean 같은 어노테이션이 붙은 클래스들을 찾아서 빈 메타데이터를 수집합니다.
그 다음에 실제 빈 인스턴스들을 생성 후 @Autowired를 통해서 의존성 주입을 처리하고, @PostConstruct나 InitializingBean 같은 초기화 메소드들을 순서대로 실행합니다.
마지막에 ContextRefreshedEvent가 발생하면서 모든 빈이 준비 완료되었다는 신호를 보냅니다.

소멸은 언제 되냐면, 애플리케이션이 종료될 때나 개발자가 직접 context.close()를 호출할 때입니다. JVM이 종료 신호를 받으면 Spring Boot가 Graceful Shutdown을 수행하면서 @PreDestroy, DisposableBean, @Bean의 destroyMethod 순서로 정리 작업을 실행해요.
실무에서는 Spring Boot가 이런 복잡한 과정들을 모두 자동화해주기 때문에 개발자가 Container 생명주기를 직접 관리할 일은 거의 없습니다.

---

## 1-2. BeanFactory와 ApplicationContext의 차이점과 ApplicationContext의 추가 기능들을 설명해주세요.

### **답변:**

### **BeanFactory와 ApplicationContext의 핵심 차이점:**

**BeanFactory**는 Spring IoC 컨테이너의 **가장 기본적인 인터페이스**로, **Lazy Loading 방식**으로 동작합니다. Bean을 실제로 요청(`getBean()`)할 때까지 생성하지 않아 메모리 효율적이지만 기본적인 DI 기능만 제공합니다.

**ApplicationContext**는 BeanFactory를 상속받는 `ListableBeanFactory`, `HierarchicalBeanFactory`를 상속받아 모든 기능을 포함하면서, **6개의 핵심 인터페이스를 통해 확장된 기능**을 제공합니다. **Eager Loading 방식**으로 컨테이너 시작 시 모든 Singleton Bean을 미리 생성하여 런타임 성능이 우수하고 시작 시점에 설정 오류를 발견할 수 있습니다.

### **ApplicationContext의 6가지 핵심 기능:**

**첫 번째, 환경 관리 기능(EnvironmentCapable)** 입니다. Profile 관리와 Property 값 조회가 가능하며, `@Profile("dev")` 같은 환경별 Bean 등록과 설정값 접근을 제공합니다.

**두 번째, 확장된 Bean 조회 기능(ListableBeanFactory)** 입니다. 모든 Bean 목록 조회, 특정 타입의 모든 Bean 조회, 등록된 Bean 개수 확인이 가능합니다.

**세 번째, 계층적 구조 지원(HierarchicalBeanFactory)** 입니다. 부모-자식 관계를 지원하여 공통 설정은 부모에, 특화 설정은 자식에 분리할 수 있습니다.

**네 번째, 국제화 지원(MessageSource)** 입니다. 다국어 메시지 처리가 가능하며, Locale에 따라 적절한 메시지를 자동으로 선택하여 반환합니다.

**다섯 번째, 이벤트 시스템(ApplicationEventPublisher)** 입니다. 애플리케이션 이벤트 발행과 `@EventListener`를 통한 이벤트 구독을 지원하여 컴포넌트 간 느슨한 결합을 제공합니다.

**여섯 번째, 리소스 로딩 기능(ResourcePatternResolver)** 입니다. `classpath:`, `file:`, `http:` 등 다양한 프로토콜을 지원하고 패턴 매칭을 통한 다중 리소스 조회를 제공합니다.

### **내부 동작 원리:**

ApplicationContext는 시작 시 **refresh() 메서드를 통해 초기화**되며, 내부적으로 **실제 Bean 저장소 역할을 하는 BeanFactory**를 가지고 있습니다. **@Configuration 클래스 처리**와 **@Autowired 등의 어노테이션 처리**를 담당하는 후처리기들이 동작하여 어노테이션 기반 설정을 지원합니다.

실무에서는 보통 Spring boot를 사용하기때문에 이러한 **6가지 통합된 기능**과 **타입 기반 Bean 조회**, **환경별 설정 관리**, **이벤트 기반 아키텍처** 등의 장점 때문에 BeanFactory보다는 **ApplicationContext를 주로 사용**합니다.

---

## 2. Spring의 다양한 Bean Scope(Singleton, Prototype, Request, Session 등)의 특징과 내부 구현 방식을 설명하고, Singleton Bean에서 Prototype Bean을 주입받을 때 발생하는 문제와 해결 방법을 설명해주세요.


### **답변:**

1. Singleton (기본값)

- 스프링 컨테이너당 하나의 인스턴스만 생성
- 애플리케이션 시작 시 생성되어 종료 시까지 유지
- 메모리 효율적이고 성능상 유리

2. Prototype

- 매번 새로운 인스턴스를 생성
- 컨테이너는 생성 후 관리하지 않음 (생명주기 관리 책임이 클라이언트에게)
- 상태를 가지는 Bean에 적합

3. Request (웹 환경)

- HTTP 요청당 하나의 인스턴스 생성
- 요청 완료 시 소멸

4. Session (웹 환경)

- HTTP 세션당 하나의 인스턴스 생성
- 세션 종료 시 소멸

### 내부 구현 방식

Spring은 BeanFactory와 ApplicationContext를 통해 Bean Scope를 관리합니다.

- Singleton: DefaultSingletonBeanRegistry에서 singletonObjects Map으로 관리
- Prototype: 매번 createBean() 메서드를 호출하여 새 인스턴스 생성
- Request/Session: RequestScope, SessionScope 클래스가 HTTP 요청이나 세션과 연동하여 관리

### Singleton Bean에서 Prototype Bean을 주입받을 때 발생하는 문제
prototype bean에서 singleton bean을 주입받아 사용하면 의도한대로 prototype bean은 매번 생성되고 sigleton bean은 같은 객체를 사용해 문제가 없지만,
반대의 경우에는 prototype bean이 매번 새로 생성되는게 아닌 같은 인스턴스를 계속해서 참조하는 문제가 있습니다.

그경우에는 prototype bean에 `@Lockup` 어노테이션 사용이나, `proxyMode = ScopedProxyMode.TARGET_CLASS`를 사용하는 방식, 또는 `ApplicationContext`를 사용한 `getBean()` 메서드를 사용해 새 인스턴스를 생성하는 작업을 할 수 있습니다.

---

## 3. Spring MVC의 DispatcherServlet이 요청을 처리하는 전체 과정을 설명하고, HandlerMapping, HandlerAdapter, ViewResolver의 역할을 설명해주세요.

DispatcherServlet은 Spring MVC 프레임워크에서 클라이언트의 HTTP 요청을 처리하는 핵심 컴포넌트입니다. 클라이언트의 요청을 받아 적절한 Controller에게 위임하고, View를 통해 응답을 생성하여 클라이언트에게 반환하는 역할을 합니다. 주요 처리 과정은 다음과 같습니다:
1. 클라이언트 요청:
   웹 브라우저 등 클라이언트가 HTTP 요청을 DispatcherServlet으로 보냅니다.
2. DispatcherServlet 수신:
   DispatcherServlet이 요청을 받습니다.
3. HandlerMapping:
   DispatcherServlet은 HandlerMapping을 사용하여 요청 URL과 일치하는 Controller를 찾습니다.
4. HandlerAdapter:
   DispatcherServlet은 HandlerAdapter를 통해 컨트롤러를 실행하고 그 결과를 받습니다.
5. Controller 실행:
   찾은 Controller를 실행하여 요청을 처리합니다. 이 과정에서 서비스 로직, 데이터베이스 접근 등이 포함될 수 있습니다.
6. ViewResolver:
   DispatcherServlet은 ViewResolver를 사용하여 처리 결과를 기반으로 어떤 View를 사용할지 결정합니다.
7. View 렌더링:
   선택된 View는 요청에 대한 결과를 렌더링합니다. (예: HTML 페이지 생성)
8. 응답:
   렌더링된 결과를 클라이언트에게 HTTP 응답으로 반환합니다.

> **RestController의 특징**
> 
> RestController는 `@ResponseBody`가 기본적으로 적용되어 있어서 ViewResolver를 사용하지 않습니다. 대신 `HttpMessageConverter`가 메소드의 반환값을 JSON, XML 등으로 직렬화하여 클라이언트에게 바로 전달합니다.