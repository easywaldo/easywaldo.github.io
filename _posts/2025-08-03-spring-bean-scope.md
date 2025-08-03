---
layout: single
title: "SpringFramework Bean Scope"
date: 2025-08-03 09:50:00 +0900
categories:
  - SpringFramework
tags:
  - Bean Scope
  - SpringFramework
toc: true
toc_sticky: true
comments: true
---

# Bean Scope 는 무엇인가?
빈 정의를 생성하면 해당 빈 정의에 정의된 클래스의 실제 인스턴스를 스프링 컨테이너가 생성을 하게 됩니다.
이때 빈 정의에서 생성된 객체의 범위도 제어할 수 있습니다.

## Bean Scope 종류
스프링에서 제공하는 빈의 범위는 다음과 같습니다.
- **Singleton**: 기본 범위로, 스프링 컨테이너당 하나의 인스턴스만 생성됩니다.
- **Prototype**: 요청할 때마다 새로운 인스턴스를 생성합니다.
- **Request**: HTTP 요청당 하나의 인스턴스를 생성합니다.(웹 기반 Spring 컨텍스트에서만 유효합니다)
- **Session**: 단일 Bean 정의의 범위를 HTTP 라이프사이클로 지정합니다.(웹 기반 Spring 컨텍스트에서만 유효합니다)
- **application**: ServletContext 당 하나의 인스턴스를 생성합니다.(웹 기반 Spring 컨텍스트에서만 유효합니다)
- **Websocket**: WebSocket 세션당 하나의 인스턴스를 생성합니다.(웹 기반 Spring 컨텍스트에서만 유효합니다)


### 싱글톤 스코프와 프로토타입 스코프
![싱글톤 스코프 와 프로토타입 스코프]({{ site.baseurl }}/assets/images/singleton_and_prototype_scope.png)


#### 예제 코드
```kotlin

class SampleBean {
    fun greet(): String {
        return "Hello from SampleBean!"
    }

    fun getHashCode(): String {
        return "hashCode: ${super.hashCode()}"
    }
}

@Configuration
class BeanConfig {
    @Bean(name = ["sampleBean"])
    @Scope("singleton")
    fun singletonBean(): SampleBean {
        return SampleBean()
    }

    @Bean(name = ["sampleBean2"])
    @Scope("prototype")
    fun prototypeBean(): SampleBean {
        return SampleBean()
    }
}


fun main(args: Array<String>) {
    val applicationContext = runApplication<DemoApplication>(*args)

    val singletonBean1 = applicationContext.getBean("sampleBean") as com.example.demo.config.SampleBean
    val singletonBean2 = applicationContext.getBean("sampleBean") as com.example.demo.config.SampleBean
    println("Singleton Bean 1: ${singletonBean1.getHashCode()}")
    println("Singleton Bean 2: ${singletonBean2.getHashCode()}")

    val prototypeBean1 = applicationContext.getBean("sampleBean2") as com.example.demo.config.SampleBean
    val prototypeBean2 = applicationContext.getBean("sampleBean2") as com.example.demo.config.SampleBean
    println("Prototype Bean 1: ${prototypeBean1.getHashCode()}")
    println("Prototype Bean 2: ${prototypeBean2.getHashCode()}")
}

```

#### 실행 결과
```shell
Singleton Bean 1: hashCode: 1952627275
Singleton Bean 2: hashCode: 1952627275
Prototype Bean 1: hashCode: 103402417
Prototype Bean 2: hashCode: 375039034
```


### Request, Session, Application, Websocket 범위
어노테이션으로 위의 범위들을 정의할 수 있습니다.

```kotlin
@RequestScope
@Component
class LoginAction {
	// ...
}

@SessionScope
@Component
class UserPreferences {
	// ...
}

@ApplicationScope
@Component
class AppPreferences {
	// ...
}
```


### 종속성에 의한 Bean Scope
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```

위와 같이 정의가 된 UserManager Bean 은 싱글톤 스코프로 정의가 되어 있어
UserReferences Bean 또한 UserManager Bean 인스턴스가 생성이 되는 시점에
오직 한번만 주입이 되게 됩니다. 이는수명이 짧은 범위인 빈을 수명이 긴 범위인 빈에 주입할 때 원하는 동작이 아닙니다

따라서 아래와 같이 UserManager Bean에 UserPreferences Bean을 Proxy 로써 주입을 하여 UserManager 객체에서 UserPreferences 메서드를 호출할 때 실제로는 프록시에서 메서드를 호출합니다. 그런 다음 프록시는 HTTP에서 실제 객체를 가져 오고 검색된 실제 객체에 메서드 호출을 위임합니다

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
	<aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```


___
참고문서
https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-singleton
