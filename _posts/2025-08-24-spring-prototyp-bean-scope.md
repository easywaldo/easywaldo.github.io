---
layout: single
title: "SpringFramework Bean Prototype Scope"
date: 2025-08-04 00:50:00 +0900
categories:
  - SpringFramework
tags:
  - Bean Scope
  - Prototype Scope
  - SpringFramework
toc: true
toc_sticky: true
comments: true
---

# Prototype Scope 에 대해서 자세히 알아보기
> 스프링 교과서의 내용을 기반으로 참고 해보았습니다. (길벗출판사)

만약 강좌등록서비스가 존재하고
이를 스프링 빈으로 등록한다고 가정해봅시다.
이때 강좌등록서비스는 싱글톤 빈으로 정의를 합니다.
    
```kotlin
@Service
class CourseRegistrationService {
    // ...
}
```

그리고 강좌등록서비스는 여러 강좌들을 조회,수정,삭제하는 기능을 가지고 있다고 가정해봅시다.
```kotlin
@Service
class CourseRegistrationService {
    fun registerCourse(course: Course) {
        // ...
    }

    fun updateCourse(course: Course) {
        // ...
    }

    fun deleteCourse(courseId: Long) {
        // ...
    }
}
````

이때 강의수강평을 CourseCommentProcessor 라는 클래스에
책임을 위힘하여 CourseSerivce 는 수강평을 등록한다고 가정해봅시다.

이때 우리는 CourseCommentProcessor 를 빈으로 관리를 할 것인지
고민을 할 수 있습니다. 사실 근본적으로 스프링에서는 어떤 객체를 빈으로 등록을 하기 전에 이 객체를 빈으로 등록하여 인스턴스 관리를 해야 하는지에 대한 고민을 하는 것이 중요 합니다.

이유는 해당 객체의 생명주기와 의존성 및 상태 관리를 어떻게 할 것인지에 대한 명확한 이해가 필요하기 때문입니다.

만약 CourseCommentProcessor 를 싱글톤 빈으로 등록을 한다면
CourseCommentProcessor 인스턴스 역시 동일한 인스턴스를 얻게 될 것이고 이때 여러 스레드에서는 CourseComment 에 대한 인스턴스 역시
동일한 인스턴스를 공유하게 될 확률이 높아지게 됩니다.


```kotlin
@Component
@Scope("singleton")
class CourseCommentProcessor {
    fun processComment(comment: CourseComment) {
        // 수강평을 등록 하거나 수정을 할 수 있습니다.

        // 여러 스레드에서 이 함수를 호출을 하게 될때 동일한 CourseComment 인스턴스에 접근할 수 있는 가능성이 높아지게 됩니다.
    }
}
```

그렇게 되면 여러 학생이 동시에 수강평을 등록하거나 수정할 때, 동일한 CourseComment 인스턴스에 접근하게 되어 데이터 일관성 문제가 발생할 수 있습니다. 이러한 문제를 피하기 위해 CourseCommentProcessor 를 프로토타입 빈으로 등록하는 것이 좋습니다.

한 가지 더 고려해야 할 점은 CourseCommentProcessor 가
CourseComment 를 얻기 위해서 필요한 것이 Repository 라는 점입니다. 이때 Repository 를 얻기 위한 가장 쉬원 방법은 스프링 DI 를 이용하는 것이고 따라서 CourseCommentProcessor 또한 빈으로 되어야 한다는 점입니다.

```kotlin
@Component
@Scope("prototype")
class CourseCommentProcessor(
    private val courseCommentRepository: CourseCommentRepository    // 스프링 DI를 통해 주입받습니다.
) {

    fun processComment(comment: CourseComment) {
        // 수강평을 등록 하거나 수정을 할 수 있습니다.

        // 여러 스레드에서 이 함수를 호출을 하게 될때 동일한 CourseComment 인스턴스에 접근할 수 있는 가능성이 높아지게 됩니다.
    }
}
```

이렇게 하면 CourseCommentProcessor 의 각 인스턴스는 독립적으로 존재하게 되어, 여러 스레드가 동시에 수강평을 처리할 때도 서로 간섭하지 않게 됩니다. 즉, 각 요청마다 새로운 CourseCommentProcessor 인스턴스가 생성되어 데이터 일관성 문제를 방지할 수 있습니다.

그러나 아래와 같이 @Autowired 를 사용하여 CourseCommentProcessor 를 주입받는다면, CourseRegistrationService 인스턴스가 최초 싱글톤 빈으로 생성이 뙬때 CourseCommentProcessor 또한 최초 1회 주입이 되어 동일한 인스턴스를 공유하게 됩니다.

```kotlin
@Service
class CourseRegistrationService {
    @Autowired
    private lateinit var courseCommentProcessor: CourseCommentProcessor

    fun registerCourse(course: Course) {
        // ...
    }

    fun updateCourse(course: Course) {
        // ...
    }

    fun deleteCourse(courseId: Long) {
        // ...
    }
}
````

이러한 문제를 해결하기 위해서는 스프링 컨텍스트를 통해서 CourseCommentProcessor 의 새로운 인스턴스를 요청해야 하는데 이를 위해서는 ApplicationContext 를 주입받아 사용하면 됩니다.

```kotlin
@Service
class CourseRegistrationService(
    private val applicationContext: ApplicationContext
) {
    fun registerCourse(course: Course) {
        val courseCommentProcessor = applicationContext.getBean(CourseCommentProcessor::class.java)
        // ...
    }

    fun updateCourse(course: Course) {
        // ...
    }

    fun deleteCourse(courseId: Long) {
        // ...
    }
}
```

위와 같이 하게 되면 각 요청마다 새로운 CourseCommentProcessor 인스턴스를 얻을 수 있게 되어, 여러 스레드가 동시에 수강평을 처리할 때도 서로 간섭하지 않게 됩니다.

하지만 이러한 방식은 매번 새로운 인스턴스를 생성하기 때문에 성능에 영향을 줄 수 있습니다. 따라서 실제 서비스에서는 이러한 점을 고려하여 적절한 빈 스코프를 선택하는 것이 중요합니다.

또한 프로토타입 스코프의 경우 변경 가능한 속성을 포함 할 수 있으므로 어떠한 기능을 수행함에 있어서 부수효과(부작용)을 발생시킬 수 있습니다. 따라서 프로토타입 빈을 사용할 때는 이러한 부작용을 최소화하기 위한 설계가 필요합니다.