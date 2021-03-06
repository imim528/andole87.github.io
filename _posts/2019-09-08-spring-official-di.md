---
published: true
layout: single
title: "스프링 공식문서 읽기 - DI편"
category: Spring
comments: true
---

우아한테크코스에서 스프링을 처음 사용해봤다. 짧은 기간동안 스프링으로 미션을 진행해야 했다. 아쉬운 부분이 많아 방학을 기다려 공식문서를 읽어보았다.  
공식 문서에는 [`Core Technologies`](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html) 섹션이 있었고, 관심있는 주제들이 많았다. 양이 상당해 필요한 부분을 집중적으로 보았다. 혼자만 읽기에는 아까워서 ~~발번역~~을 남겨본다.  

다음 내용은 공식 문서의 내용을 그대로 옮긴 것이 아니며, 임의로 편집 수정했음을 미리 밝힌다.  

# Introduction to the Spring IoC Container and Beans

`IoC`는 `DI`로도 알려져 있다. 객체의 생성자 인자, 팩토리 메서드 인자, setter메서드를 통해서 의존성을 정의하면,  스프링 컨테이너가 의존성을 주입한다. 이 과정은 근본적으로 제어를 역전시킨다. 그래서 스프링 컨테이너를 `Inversion of Control Container`라고도 부른다.

IoC Container의 토대는 `org.springframework.beans`, `org.springframework.context` 패키지에 있다.

`org.springframework.beans.BeanFactory` 인터페이스는 어떤 타입의 객체라도 빈으로 설정할 수 있는 메카니즘을 제공한다.

`ApplicationContext`는 `BeanFactory`의 하위 인터페이스다. `ApplicationContext`는 `BeanFactory`에 다음 기능을 추가한다.

- `Spring AOP`와의 통합을 제공

- `MessageResource` 핸들링

- 웹 어플리케이션에서 사용되는 `WebApplicationContext`처럼 Application-layer의 구체적인 context를 제공한다. 

요약하면, `BeanFactory`는 프레임워크의 기본 설정과 기능을 제공한다. `ApplicationContext`는 이에 구체적인 비즈니스 기능을 추가한다.

스프링에서는 어플리케이션의 중추를 이루는 객체나 스프링 컨테이너가 관리하는 객체를 `Bean`이라고 부른다. Bean은 인스턴스화되어 있고 의존성이 조립되어 있는, Spring이 관리하는 객체다. 컨테이너는 Bean들과 이 Bean들이 연관된 의존성들이 reflection된 메타데이터 설정들을 사용하게 된다.

# Container Overview

`ApplicationContext`는 스프링 컨테이너를 대표하는 인터페이스다. 객체 생성, 설정, 의존성 조립을 책임진다.

컨테이너는 빈의 metadata를 읽어서 객체 생성, 설정, 조립 등을 수행한다. 설정 정보는 XML, Annotation, Java Code 등으로 설정할 수 있다. 

`ApplicationContext`는 몇 가지 구현체가 있는데, `ClassPathXmlApplicationContext`, `FileSystemXmlApplicationContext`가 흔히 사용된다.

XML이 스프링 설정을 위해 전통적으로 사용된 형식은 맞지만, 어노테이션이나 자바 코드로 메타데이터를 설정할 수 있다. 

다음 다이어그램은 고수준에서 스프링이 동작하는 방식을 보여준다.  어플리케이션 클래스들은 `ApplicationContext`에 의해 메타데이터 설정과 통합되어 실행 가능한 어플리케이션이 된다.

![다이어그램](/assets/container-magic.png)

>  주) 본문에선 xml기반의 설정으로 ApplicationContext를 생성하는 법만을 다룬다. 여기서는 Annotation 기반 설정으로 ApplicationContext를 생성하는 법을 다뤄본다.

```java
@Component
public class A { 
  public void doSomething() {
    System.out.println("It is A.class Bean");
  }
}

@ComponentScan
public class AnnotatedContextApplication {
  public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(AnnotatedContextApplication.class);
    A bean = context.getBean(A.class);
    bean.doSomething();
  }
}
```


# Bean Overview

스프링 컨테이너는 하나 이상의 빈들을 관리한다. 이 빈들은 설정 메타데이터(XML이나 어노테이션 등)를 통해 생성된다. 컨테이너 내부에는, 빈 정의(bean definition)가 `BeanDefinition` 객체로 관리된다. `BeanDefinition`은 다음 메타데이터를 가진다.

- 패키지 기반의 클래스 이름
- 빈이 의 동작 설정 정보들. (스코프, 라이프사이클 콜백 등)
- 동작 수행에 필요한 다른 빈(의존성)의 참조
- 새로운 객체 생성에 관한 설정 정보 (예 : Connection Pool을 관리하는 빈은 Connection Pool 최대 사이즈 정보를 갖고 있다.)

---

다음 메타데이터들은 `BeanDefinition`의 설정정보로 변환되어 저장된다.

- Class
- Name
- Scope
- Constructor arguments
- Properties
- Autowiring mode
- Lazy Initialization mode
- Initialization method
- Destruction method

---

컨테이너는 빈을 어떻게 생성하는지의 메타데이터로 스스로 빈을 만들고 관리할수 있다. 더해서, `ApplicationContext`는 외부에서(유저에 의해) 생성된 객체를 빈으로 등록할 수도 있다. 

ApplicationContext의 `getBeanFactory()`는 `DefaultListableBeanFactory`를 반환한다. 이 객체는 `registerSingleton()`, `registerBeanDefinition()`메서드를 제공한다. 이 메서드를 통해 외부에서 생성된 객체들을 ApplicationContext가 관리하도록 등록할 수 있다.

런타임에 새로운 빈을 등록하거나 싱글턴 빈을 교체하거나 덮어쓸 수 있다. 그러나 공식적으로 지원되는 방식은 아니다. 어플리케이션 전반에 걸쳐 오동작이나 버그가 발생할 여지가 매우 크다.

> 유저가 직접 생성한 빈들은 최대한 빨리 컨테이너에 등록해야 한다. 다른 빈들과 연결하거나 동작하게 만들 때, NPE나 오동작의 원인이 될 수 있기 때문이다.

# Dependencies

대부분의 엔터프라이즈 어플리케이션은 하나의 객체로 이뤄지지 않는다. 가장 단순한 어플리케이션이라 하더라도, 몇 개의 객체들이 긴밀히 협력한다. 이 섹션에서는 빈들이 목표를 위해 잘 협력할 수 있도록 빈들을 어떻게 정의할지 설명한다.

### DI

`DI`는 객체의 생성자, 팩토리 메서드, setter에 의해서만 수행된다. 

DI 원칙을 잘 지킨다면 코드는 간결해지고 의존성은 낮아진다. 객체들은 자신이 필요한 객체를 찾아 헤메거나, 필요한 객체가 어디에 있는지 알 필요가 없다. 때문에 클래스들은 테스트하기 쉬워진다. 특히 의존성이 인터페이스나 추상 클래스를 기반으로 하는 경우, `stub`이나 `mock`을 통해 단위 테스트가 가능하다.

DI는 크게 두 가지 버전이 있다. 생성자를 통한 방법, setter를 통한 방법이다.

## Constructor - based

생성자를 통한 DI는 컨테이너가 생성자 인수와 함께 생성자를 호출하는 것으로 수행된다. 이것은 static factory method를 호출하는 것과 거의 비슷하다. 그래서 여기서는 생성자와 static factory method를 비슷하게 취급한다.

다음 예제는 생성자를 통해서만 DI할 수 있는 클래스를 보여준다.

```java
public class SimpleMovieLister {
  private MovieFinder movieFinder;

  public SimpleMovieLister(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
  }
}
```


별다를 것 없다는 것에 주목하라. 이 클래스는 소위 POJO라고 불리는 자바 본연의 객체다. 어떤 인터페이스도, 베이스 클래스도, 어노테이션도 없다.

### 생성자 인자 결정(constructor arguement resolution)

생성자에 주입할 인자는 인자의 타입으로 매칭된다. `BeanDefinition`에 잠재적인 ambiguos 인자가 없다면(같은 타입의 여러 빈이 없다면) 순서대로 생성자 인자에 할당해 객체를 생성한다.

그러나 `<value>true</value>`와 같이 타입 없는 값이 사용되었다면, 스프링은 타입이 무엇인지 알수 없다.  

```java
public class ExampleBean {
  private int years;
  private String ultimateAnswer;

  public ExampleBean(int years, String ultimateAnswer) {
    this.years = years;
    this.ultimateAnswer = ultimateAnswer;
  }
}
```


이 상황에서는 스프링에 `type` 속성을 제공해야 한다. 

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

생성자 인자에 인덱스를 부여해서 해결할 수도 있다. 인덱스는 0부터 시작한다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="1" value="42"/>
    <constructor-arg index="0" value="7500000"/>
</bean>
```

## Setter - based

Setter 기반 DI는 기본 생성자로 빈을 만든 후, setter호출로 의존성을 주입한다. 

```java
public class SimpleMovieLister {
  private MovieFinder movieFinder;

  public void setMovieFinder(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
  }
}
```


ApplicationContext는 생성자 기반과 setter 기반 모두 지원한다. 또한 생성자 기반 DI가 이루어진 후, setter 기반 DI로 의존성을 해결할 수도 있다.

`BeanDefinition`에 `PropertyEditor`를 연결하면 직접 의존성을 바꿀 수 있다. 그러나 대부분의 스프링 유저는 이렇게 직접(programmatically) 설정하지 않는다. XML로 설정하거나, 어노테이션으로 설정하는게 좋다. 이렇게 설정하는 것 역시 내부적으로 `BeanDefinition`으로 변환되어 ApplicationContext에 전달된다. 이렇게 해야 코드 레벨에서 의존성이 제거되고 코드 외부에서 설정을 변경하기 용이해진다.

### 생성자 DI vs setter DI

스프링 팀은 일반적으로 생성자 기반 DI를 지지한다. `Immutable`하게 시스템을 설계할 수 있는데다, 필요한 의존성들이 `NULL`이 아님을 보장할 수 있기 때문이다. 더욱이 생성자 DI는 필요한 때에 완전히 구성된 빈을 리턴할 수 있다.

조금 다른 관점이지만 많은 수의 생성자 인자는 나쁜 코드의 냄새를 풍기게 한다. 해당 빈이 너무 많은 책임을 지고 있다는 표시다. 생성자 기반 DI를 사용하면 어느 클래스가 과도한 책임을 갖고 있는지, 리팩토링이 필요한지 쉽게 느낄 수 있다.

Setter DI는 우선적으로 Optional한(_JAVA Optional 이 아닌, 여러 옵션을 가진_) 의존성에 적용되어야 한다. 그렇지 않으면 null 체크를 지겹게 해야 한다. Setter DI의 장점은 나중에 re-injecting하기 편하다는 것이다. 

## Dependency Resolution Process

컨테이너는 다음과 같이 DI를 수행한다.

설정 메타정보를 토대로 ApplicationContext가 생성된다. ApplicationContext는 필요한 빈을 생성한다.

모든 빈은 자신의 의존성을 속성정보(form of properties)나 생성자 인자 또는 factory method의 인자로 기술한다. 컨테이너는 빈이 실제로 만들어질 때, 이 의존성들을 제공(주입)한다.

모든 속성이나 생성자 인자는 실제 값이거나, 컨테이너가 관리하는 다른 빈의 참조이다.

값으로 정의된 모든 속성이나 생성자 인자는 특정 형태로부터 변환된 값이다. 스프링은 기본적으로 내장 타입을 값으로 변환할 수 있다. int, long, String, boolean ...

스프링 컨테이너는 스스로가 만들어질 때(스프링 컨테이너가 만들어질때), 각 빈의 설정을 검증한다. 그러나 **빈들이 실제로 만들어질 때까지 빈들은 설정 정보가 세팅되지 않는다.**

**싱글턴이거나 미리 생성된 빈들은 컨테이너가 생성될 때 만들어진다.** 다른 빈들은 필요할 때만 만들어진다. 

빈을 만드는 과정은 빈의 의존성에 따라 여러 빈을 만들어야 한다.(의존성의 의존성의 의존성의 ...) 이 의존성 해결 때문에, 빈들의 첫 생성이 느려질 수 있다.

#### 순환 참조

생성자 DI를 주로 사용한다면 순환참조가 발생할 가능성이 있다.

순환참조는 A가 B를 필요로 하는데, B도 A를 필요로 하는 경우 발생한다. 스프링 컨테이너는 이를 감지해서 `BeanCurrentlyInCreationException`을 발생시킨다.

해결방법 중 하나는 빈 하나를 Setter DI로 바꾸는 것이다. 아니면 모두 Setter DI로 바꾸거나.(물론 추천하진 않음) 다른 방법 중 하나는 빈 생성시 우선순위를 정해주는 것이다.

스프링은 설정 정보를 검사하여 순환참조나, 존재하지 않는 빈이 의존성으로 등록된 경우 예외를 발생시킨다. 이 과정이 ApplicationContext를 로드(어플리케이션을 시작할 때)이루어지므로 신뢰할 수 있다.

그러나 스프링은 설정과 의존성을 필요할 때까지 가능한 늦게 실행하려 한다. 즉, 로딩은 정상적으로 진행되었으나 나중에 예외를 발생시킬 수 있다는 뜻이다.

때문에 ApplicationContext는 싱글턴 빈들을 미리 생성하도록 기본 설정 되어 있다. 빈들을 미리 생성하는데 필요한 메모리 비용, 시간을 소모하는게 실제로 필요할 때 문제가 발생하는 것보다 낫기 때문이다. (그럼에도 LazyInitialize하도록 설정을 바꿀 수 있다.) 

# Bean Scopes

bean definition을 정의하면 대상 클래스의 실제 인스턴스를 만드는 "방법"(recipe)가 만들어진다. bean definition이 레시피라는 것은 중요한 개념이다. 하나의 bean definition으로 여러 인스턴스를 만들 수 있기 때문이다.

대상 객체에 연결될 여러 의존성과 설정값들은 bean definition으로 제어할 수 있다. 더해서 bean definition으로 빈의 scope도 제어할 수 있다. 이러한 접근은 빈을 직접 만드는(bake) 대신 만드는 방법을 제공한다는(choose the scope) 점에서 강력하고 유연하다.

빈은 여러가지 스코프를 가질 수 있다. 스프링에서 기본으로 제공하는 것은 6개이며, 이중 4개는 웹 어플리케이션에서만 사용 가능하다.

| Scope       | Description                                                         |
| ----------- | ------------------------------------------------------------------- |
| singleton   | (기본값) 하나의 컨테이너에 하나의 인스턴스만 존재한다.                                     |
| prototype   | 모든 빈 인스턴스에 각 하나씩 존재한다.                                              |
| request     | 각 HTTP 요청 라이프사이클당 하나의 인스턴스가 존재한다. 즉, 각각 HTTP 요청에 대해서 해당 빈을 하나씩 가진다. |
| session     | 각 HTTP Session 라이프사이클당 하나의 인스턴스가 존재한다.                              |
| application | ServletContext 라이프사이클당 하나의 인스턴스가 존재한다.                              |
| websocket   | WebSocket 라이프사이클당 하나의 인스턴스가 존재한다.                                   |

> 스프링 3.0부터 thread scope도 가능하다. 스프링 4.2부터는 transaction scope도 가능하다.



## Singleton scope

싱글턴 스코프는 스프링 빈의 기본 스코프다. 오직 하나의 공유 인스턴스만 관리되며, 컨테이너로의 모든 빈 요청에 대해 하나의 인스턴스만 반환된다.

다시 말하면 빈을 싱글턴 스코프로 정의하면 스프링 컨테이너는 빈을 **오직 한번 생성한다.** 이 싱글턴 인스턴스는 캐시에 저장되며 스프링 컨테이너는 이후의 모든 빈 요청에 대해서 **캐시에서 꺼내 반환한다.**

![singleton](/assets/singleton.png)

스프링의 싱글턴 개념과 GoF의 싱글턴 패턴은 서로 다르다. GoF의 싱글턴은 하나의 `ClassLoader`당 하나의 인스턴스를 갖도록 하드코딩이 필요하다.

스프링의 싱글턴을 가장 잘 설명하는 것은 "하나의 컨테이너당 하나의 빈이 존재하도록" (Being per-container and per-bean)이다. 즉, 클래스를 singleton scope로 정의하면,  스프링 컨테이너는 bean definition에 기술된 대로 오직 하나의 인스턴스를 생성한다.

빈을 싱글턴 스코프로 정의하려면 다음과 같이 한다. (기본값이므로 따로 설정하지 않아도 된다.)

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

어노테이션은

```java
@Bean
@Scope("singleton")
public AccountService create() {
  ...
}

@Component
@Scope("singleton")
public class AccountService {
  ...
}
```

## Prototype scope

프로토타입 스코프를 갖는 빈은 컨테이너에 요청할 때마다 인스턴스를 새로 생성한다. 다시 말하면 빈이 주입될 때나 `getBean()` 메서드가 호출될 때마다 새로 만들어진다. 

대체로 프로토타입 스코프는 빈이 상태를 가질 때(stateful) 사용한다. 상태를 가지지 않는다면 (stateless) 싱글턴 스코프가 적절하다.

![prototype](/assets/prototype.png)

프로토타입 스코프를 적용하려면 다음과 같이 한다.

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

어노테이션은

```java
@Bean
@Scope("prototype")
public AccountService create() {
  ...
}

@Component
@Scope("prototype")
public class AccountService {
  ...
}
```

프로토타입 스코프인 빈은 다른 스코프의 빈들과는 다르게, 스프링이 라이프사이클을 완전히 관리하지 않는다. 컨테이너는 빈을 생성하고, 설정값이나 의존성을 주입한 다음 클라이언트 코드에 넘긴다. 그리고 나서는 아무 것도 하지 않는다.

그러므로 컨테이너는 스코프에 관계없이 설정된 객체 생성은 언제나 실행하는 반면, 설정된 객체 소멸자(destruction lifecycle callback)은 언제나 실행되지 않는다(prototype은 실행안함)

클라이언트 코드는 반드시 프로토타입 스코프 빈을 제거해야 한다. 그렇지 않으면 프로토타입 스코프 빈이 쓸데없이 메모리를 점유할 것이다. 메모리 누수가 일어난다는 뜻이다.

만약 컨테이너가 자동으로 프로토타입 빈들을 처리하도록 하려면, `BeanPostProcessor`를 커스터마이징해야 한다. 기본 `BeanPostProcessor`는 쓸모없어진 프로토타입 빈의 참조를 갖고 있다.

### singleton beans with prototype-bean dependencies

싱글턴 빈이 의존하는 대상이 프로토타입 빈이라고 생각해보자. 싱글턴 빈은 처음에 단 한번 만들어지므로 하나의 프로토타입 빈만을 참조할 수 있다. 즉, 싱글턴 빈이 만들어질 때 주입된 프로토타입 빈이 주입되고, 새로운 프로토타입 빈은 사용되지 않는다.

싱글턴 빈이 여러 프로토타입 빈을 사용해야 할 때는 메서드 기반 DI를 사용해야 한다.

## Request, Session, Application and WebSocket Scopes

이 스코프들은 `XmlWebApplicationContext`와 같은 ApplicationContext를 사용하는 웹 어플리케이션에서만 사용할 수 있다.

만약 웹이 아닌 `ClassPathXmlApplicationContext`와 같은 ApplicationContext에 빈을 등록하려 하면 컨테이너가 `IllegalStateException` 예외를 던진다.

### Initial Web Configuration

`request`, `session`, `application`, `websocket` 스코프를 사용하려면 몇 가지 세부 설정이 필요하다.(singleton, prototype은 필요 없다)

스프링 Web MVC를 사용한다면 추가적인 어떤 설정도 필요 없다. Spring Web MVC에 포함되어 있는 `DispatcherSevlet`이 알아서 적절한 설정을 해준다.

`DispatcherServlet`은 `RequestContextListener`, `RequestContextFilter`와 완전히 같은 동작을 수행한다. HTTP 요청 객체를, 요청을 수행할 `Thread`에 바인드한다. 그리고 콜 체인에 포함되는 빈을 `request` 또는 `session` 스코프로 설정한다.

## Scoped Beans as Dependencies

스프링 컨테이너는 빈의 생성 뿐만 아니라 의존성 연결도 수행한다.(the wiring up of collaborators) 

만약 HTTP request 스코프를 갖는 빈을 더 긴 라이프사이클을 가진 빈에 주입하고 싶다면, AOP Proxy 빈을 주입하는 것도 고려할 수 있다.

> 싱글턴 빈 사이에  `<aop:scoped-proxy/> ` 를 사용할 수도 있다. 이 프록시는 serializeable하고 역시 타겟 객체를 deserialize해서 deserialize된 타겟 객체를 가져올 수 있다.
>
> 프로토타입 빈에 `type="prototype"` 대신 `<aop:scoped-proxy/>` 를 정의할 수도 있다. 모든 메서드 호출은 공유되는 프록시를 통해 이루어지며, 공유 프록시는 새로운 타겟 객체, 즉 프로토타입 빈에 요청을 위임한다.
>
> JSR 330은 이 프록시를 Provider로 부른다. 타겟 빈에 대한 메서드 호출이 있을 때마다 Provider<TargetBean>의 get()를 호출하도록 되어 있다.

> CGLIB 프록시는 오직 public 메서드의 호출만을 인터셉트할 수 있다.
>
> 주) 프록시 객체 생성에는 두가지 방법이 있다. JDK가 지원하는 DynamicProxy와 CGLIB(Code Generation Library)프록시다. 스프링 AOP는 기본으로 DynamicProxy를 사용하며 스프링은 CGLIB를 사용한다. Interface기반으로 동작하는 DynamicProxy에 비해 CGLIB는 클래스를 대상으로 할 수도 있으며 속도면에서 더 빠르다고 한다. 

# Annotation-based Container Configuration

> XML vs Annotation
> 어노테이션 기반 설정과 xml 기반 설정 중 무엇이 더 나을까? 간단한 대답은 "그때그때 다르다" 이다.  
> 각각 장점과 단점이 있다. 개발자들은 그때그때 장점을 활용할 수 있게 선택하면 된다.   
> 어노테이션은 적은 노력으로 많은 설정들을 간결하게 적용할 수 있다.  
> XML은 컴포넌트를 조립하기 위해 소스코드 어디에도 추가적인 작업이 필요 없다. 재 컴파일하거나 소스코드를 수정할 필요가 없다는 뜻이다. 그래서 어떤 개발자들은 어노테이션 기반으로 설정하면 더이상 POJO가 아닌데다, 설정들이 분산되므로 통합 관리하기 어려울 것이라고 주장한다.
>   
> 둘 중 무엇을 선택하든지, 심지어 섞어 쓰던지 스프링은 적절히 설정을 주입할 수 있다.

> 애너테이션 주입은 XML 주입 전에 실행된다. 그래서 XML 설정이 애너테이션 설정을 덮어쓰게 된다.

## @Required

@Required 애너테이션은 다음과 같이 setter 메서드에 사용된다. 

```java
public class SimpleMovieLister {
  private MovieFinder movieFinder;

  @Required
  public void setMovieFinder(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
  }
}
```

이 애너테이션은 영향을 받는 빈 속성이 설정 타임(configuration time)에 반드시 생성되어 있어야 함을 의미한다. 영향을 받는 빈이 존재하지 않으면 컨테이너는 예외를 발생시킨다. 때문에 빈이 eager하게 생성되도록 하며, 주입 실패를 배제하고 나중에 `NPE`가 발생하지 않음을 보장한다.

> 스프링 5.1부터 @Required는 공식적으로 지원이 중단되었다. 필요한 설정은 생성자 주입으로 설정하길 권장한다.



## @Autowired

@Autowired 애너테이션은 생성자, 메서드(setter가 아니어도), 필드에 사용 가능하다.

```java
public class MovieRecommender {
  @Autowired
  private final CustomerPreferenceDao customerPreferenceDao;

  @Autowired
  public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
    this.customerPreferenceDao = customerPreferenceDao;
  }

  @Autowired
  public void setCustomerPreferenceDao(CustomerPreferenceDao customerPreferenceDao) {
    this.customerReferenceDao = customerReferenceDao;
  }

  @Autowired
  public void prepare(CustomerPreferenceDao customerPreferenceDao) {
    this.customerReferenceDao = customerReferenceDao;
  }
}
```

> 스프링 4.3부터 생성자가 하나인 경우 @Autowired 애너테이션을 사용할 필요 없다. 
>
> 생성자가 여럿인 경우, 적어도 하나에 @Autowired를 붙여 컨테이너가 무엇을 사용할지 알려줘야 한다.

> 타겟 컴포넌트의 타입과 주입할 빈의 타입이 같은지 확인해야 한다. 그렇지 않으면 런타임에 DI가 실패한다.
>
> 보통 XML 기반 설정은 ClassPath 스캔을 진행하며 세부 타입(Concrete Type)을 알 수 있다. 그러나 @Bean 애너테이션이 붙은 팩토리 메서드는 리턴 타입을 확실히 해야 한다. 몇 가지 인터페이스 체인을 구현한 컴포넌트는 가장 구체적인 타입을 명시하는게 좋다. (적어도 타겟 컴포넌트와 같도록)

@Autowired로 특정 타입을 만족하는 모든 빈을 받을 수도 있다. 배열과 컬렉션 모두 가능하다.

```java
public class MovieRecommender {
  @Autowired
  private MovieCatalog[] movieCatalogs;

  @Autowired
  private List<MovieCatalog> movieCatalogList;

  @Autowired
  private Set<MovieCatalog> movieCatalogSet;
}
```

Map 컬렉션으로도 받을 수 있다. 이 경우 Map<String, TargetBean>, Map<BEAN_NAME, BEAN> 으로 채워진다.

```java
public class MovieRecommender {
  @Autowired
  private Map<String, MovieCatalog> movieCatalogs;
}
```

Optional<T> 로 주입받을 수도 있다.

```java
public class SimpleMovieLister {
  @Autowired
  public void setMovieFinder(Optional<MovieFinder> movieFinder) {
      ...
  }
}
```

스프링 5.0부터는 @Nullable 애너테이션을 사용할 수 있다.

```java
public class SimpleMovieLister {
  @Autowired
  public void setMovieFinder(@Nullable MovieFinder movieFinder) {
      ...
  }
}
```

> @Autowired, @Inject, @Value, @Resource 애너테이션은 `BeanPostProcessor`의 구현체가 제어한다. 이 말은 `BeanPostProcessor`에는 위 애너테이션을 사용할 수 없다는 뜻이다. 커스터마이징한 `BeanPostProcessor`에는 @Bean이나 XML 설정으로 완전한 빈을 만들 수 있도록 설정해줘야 한다.

## @Primary

Autowiring은 타입에 따라 수행되기 때문에, 대상 객체가 여럿일 수 있다. @Primary 애너테이션으로 이 중에서 우선 주입해야 할 빈을 지정할 수 있다.

@Primary 애너테이션은 특정 빈이 우선 주입되어야 함을 가리킨다.

```java
@Bean
@Primary
public MovieCatalog first() {
  ...
}

@Component
@Primary
public class MovieCatalog {
  ...
}
```



## @Qualifier

@Primary는 빈들의 우선순위가 모든 곳에 대해 똑같이 결정되어 있을때는 유용하다. 그러나 다양한 곳에서 서로 다른 빈들이 우선 주입되어야 한다면 @Qualifier 애너테이션을 사용한다.

```java
@Bean("first")
public MovieCatalog first() {
  ...
}

@Component("first")
public class MovieCatalog {
  ...
}

public class MovieRecommender {
  @Autowired
  @Qualifier("first")
  private MovieCatalog movieCatalog;
  ...
}
```

> 주입받아야할 빈의 qualify 기준이 빈 이름이라면, @Qualifier 애너테이션을 사용할 필요 없다. 단순히 이름으로 구분하면 된다. 필드 이름이나 파라미터 이름으로 빈들을 우선 선택할 수 있다.

커스텀 qualifier 애너테이션을 만들 수도 있다. 새로운 애너테이션에 @Qualifier 애너테이션을 추가하면 된다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {
    String value();
}

public class MovieRecommender {
    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
      this.comedyCatalog = comedyCatalog;
    }
}
```


## Epilogue

모든 내용을 담지 않았다. 중간중간 XML 예제는 건너뛰었고 상식적인 내용도 스킵했다. 나름 중요하다고 생각한 내용을 정리했다.  
정리하고 보니 코드가 많지 않아 아쉽다. 다음 포스트에는 위 내용으로 코드 예제 몇개를 정리해보겠다.
