---
layout: single
title: "이벤트소싱과 마이크로서비스 아키텍쳐 스터디 제 3장"
date: 2025-08-02 16:00:00 +0900
categories:
  - Event Sourcing
tags:
  - DDD
  - Event Sourcing
  - Architecture
toc: true
toc_sticky: true
comments: true
---

### 제 3장: 이벤트소싱

> 이벤트소싱과 마이크로서비스 아키텍처 성공적인 이벤트 기반 시스템 구축하기 - 손경덕 지음  
> 위의 서적을 기반으로 스터디한 내용을 정리하였습니다.

#### 이벤트란 무엇인가?
이벤트는 시스템에서 발생한 중요한 사건이나 상태 변화가 기록된 하나의 사실로써의 의미를 가집니다.

#### 감사와 이력을 예시로 설명 해본다면? (단일테이블 방식)
Product (상품) 엔터티에 등록일시/등록자, 변경일시/변경자, 삭제일시/삭제자 라는 6개의 속성이 있는 경우 해당 속성들을 이용하여 누가 언제 변경을 했는지는 알수 는 있지만 무엇을 바꿨는지에 대한 정보는 여전히 알 수 없습니다. 무엇을 변경을 했는지 알기 위해서는 이전 상태를 분리해서 기록한 다음 필요할 때 현재 상태와 비교해야 합니다.

```table
| SEQ | 상품번호   | 가격  | 등록일시      | 등록자 | 변경일시      | 변경자 | 삭제일시 | 삭제자 |
|-----|----------|------|------------|-------|------------|------ |--------|------|
| 1   | P001     | 1000 | 2025-08-02 | user1 | 2025-08-02 | user1 |        |      |
| 1   | P002     | 2000 | 2025-08-02 | user1 | 2025-08-02 | user1 |        |      |
| 1   | P003     | 3000 | 2025-08-02 | user1 | 2025-08-02 | user1 |        |      |
| 2   | P001     | 1500 | 2025-08-02 | user1 | 2025-08-02 | user1 |        |      |

```

위의 테이블에서와 같이 상품코드 P001 의 경우 SEQ 1 인 시점에서 최초 등록이 된 이후
SEQ 2 인 시점에서 상품 가격이 변경이 됨을 알 수 있습니다.

위와 같이 하였을 경우 최신 상태를 조회하기 위해서는
SQL문이 복잡해질 수 밖에는 없습니다.
또한 상품별로 SEQ 에 대한 MAX 값을 조회하여 증가시키는 코드도 필요하겠고요.

#### 상태와 이력을 분리해본다면?
````table
상품정보 테이블
| 상품번호   | 가격  |

이 력테이블
| SEQ | 상품번호   | 가격  | 등록일시      | 등록자 | 변경일시      | 변경자 | 삭제일시 | 삭제자 |
````

위와 같이 하였을 경우의 장점으로는 최신 상태를 조회하기 위한 SQL 문이 단순해지는 점이 있습니다.
단점으로는 2개의 테이블에 대한 데이터 접근 코드가 생기게 되어 코드가 증가하는 점이 있습니다.


#### 변경 값
앞선 2가지의 방법은 변경하지 않은 속성도 모두 포함한 데이터의 전체 복사본을 저장하므로 효과적인 데이터베이스
저장 방식이 아닙니다.
따라서 변경된 속성만을 저장하는 방법을 이용하게 되면 데이터베이스도 효과적으로 사용할 수 있고 모든 속성의 값을 비교할 필요도 없습니다.

##### 변경한 속성 목록을 생성하는 로직은 어느 계층에 구현할까?
- 어플리케이션 계층?
: 이곳에 구현을 하게 되면 도메인 객체를 어플리케이션 계층이 자세하게 알고 있어야 합니다.
- 엔터티?
: 엔터티가 변경 요청에 포함된 속성들에 대한 지식을 가지고 있으므로 적합한 후보이긴 합니다.
하지만 불변식 유지 책임을 가진 애그리게이트와 비교할 필요가 있습니다.
- 애그리게이트?
: 에그리게이트는 변경 요청에 포함된 속성들을 소유하고 있는 엔터티와 값 객체를 알고 있으므로 불변식을 유지하는 최적의 후보입니다.
- 리포지토리?
: 리포지토리는 변경된 값을 데이터베이스에 기록하는 책임을 가진 것이고 변경 값 목록을 생성하는 것은 관계가 없습니다.


#### 변경한 속성 목록의 예시
```json
{
  "employeeId": 1,
  "seq": 1,
  "delta": {
    "employeeId": 1,
    "name": "홍길동",
    "address": {
      "street": "서울시 강남구 역삼동",
      "city": "서울",
      "state": "서울",
      "zip": "12345"
    },
    "category": "전자제품",
    "createdAt": "2025-08-02T10:00:00Z",
    "createdBy": "amdin1",
  }
}
{
  "employeeId": 1,
  "seq": 2,
  "delta": {
    "address": {
      "street": "서울시 강남구 삼성동",
      "city": "서울",
      "state": "서울",
      "zip": "12111"
    },
    "updatedAt": "2025-08-02T11:00:00Z",
    "updatedBy": "admin2"
  }
}
```

위와 같이 변경이력을 바탕으로 이전의 값과 변경된 값을 비교하여 값이 다르다면 변경된 값을 생성하여
Repository 에 저장을 하게 되는데 `Value Object`인 경우에는 동등성 비교 로직이 필요 합니다.

#### 값객체 동등성 비교
```kotlin
data class Address(
    val street: String,
    val city: String,
    val state: String,
    val zip: String
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is Address) return false

        return street == other.street &&
               city == other.city &&
               state == other.state &&
               zip == other.zip
    }

    override fun hashCode(): Int {
        return Objects.hash(street, city, state, zip)
    }
}
```

#### 변경 값 목록을 생성하는 로직 (Application Service)
```kotlin
class EmployeeAppService {
  private val employeeRepository: EmployeeRepository
  private val employeeHistoryRepository: EmployeeHistoryRepository

  fun changeEmployee(employeeId: Long, changes: Map<String, String>) {
    val employee = employeeRepository.findById(employeeId)
        ?: throw EmployeeNotFoundException("Employee not found with id: $employeeId")

    val affectedFields = mutableMapOf<String, Any>()
    changes.forEach { (key, value) ->
        when (key) {
            "name" -> if (employee.name != value) affectedFields["name"] = value
            "address" -> {
                // Address는 값 객체이므로 비교 로직이 필요
                val newAddress = parseAddress(value)
                if (employee.address != newAddress) affectedFields["address"] = newAddress
            }
        }
    }

    if (affectedFields.isNotEmpty()) {
        employeeHistoryRepository.save(EmployeeHistory(employeeId, affectedFields))
        employeeRepository.save(employee.applyChanges(affectedFields))
    }
  }
}
```

#### 변경 값 목록이 점점 많아지고 복잡해진다면?
도메인 이벤트를 도입함으로써 변경의 단위를 비즈니스 처리 과정에서 발생한 결과로 정의 할 수 있습니다.
이벤트는 사용자가 시스템에 무엇인가 처리하도록 시스템에 요청한 것임을 알 수 있는 힌트이면서 변경이 발생한 이유임을 알 수 있습니다.

- 도메인 이벤트 예시
```json
{
  "eventType": "EmployeeAddressChanged",
  "employeeId": 1,
  "oldAddress": {
    "street": "서울시 강남구 역삼동",
    "city": "서울",
    "state": "서울",
    "zip": "12345"
  },
  "newAddress": {
    "street": "서울시 강남구 삼성동",
    "city": "서울",
    "state": "서울",
    "zip": "12111"
  },
  "changedAt": "2025-08-02T11:00:00Z",
  "changedBy": "admin2"
}
```

#### 이벤트 발생의 원자성
에그리게이트가 비즈니스 요청을 받아 이를 처리 하고 에그리게이트에서 발생한 이벤트를 데이터베이스에 기록하는 일련의 과정을 하나의 트랜잭션으로 처리 해야 합니다.

따라서 아래와 같은 흐름으로 처리를 할 수 있습니다.

##### 커맨드 핸들러 → 에그리게이트 → 이벤트 저장까지 하나의 트랜잭션
- 커맨드 핸들러는 에그리게이트에 커맨드 전달
- 에그리게이트는 도메인 이벤트 생성
- 생성된 도메인 이벤트는 트랜잭션 안에서 저장소에 기록
- 이 저장이 완료되어야 커맨드 핸들러가 성공 응답을 반환함

