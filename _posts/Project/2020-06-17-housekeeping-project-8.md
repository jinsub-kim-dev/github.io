---
layout: post
title: "토이 프로젝트(8) : JPA에서의 생성 및 수정시간 생성 자동화"
date: 2020-06-17 22:45:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring JPA
---
           
## 생성시간, 수정시간

데이터베이스에 저장되는 Entity는 생성시간과 수정시간이라는 정보를 가지고 있다. Entity가 언제 생성되었는지 그리고 언제 수정되었는지에 대한 정보는 데이터 디버깅과 서비스 유지보수에 중요한 역할을 하게 된다. 모든 Entity에 대해 매번 해당 정보를 입력하고 수정하는 작업은 코드의 질을 떨어뜨리고 차후 유지보수를 어렵게 만든다. JPA에서는 Auditing이라는 기능을 사용하면 Entity가 생성되거나 수정될 때 자동으로 시간 정보를 자동으로 변경할 수 있다.

## JPA Auditing으로 생성 및 수정시간 자동화

먼저 `base.entity` 패키지에 `BaseTimeEntity`라는 이름의 클래스를 생성한다.

```java
package com.jinsub.housekeeping.base.entity;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime modifiedAt;
}
```

 * **`@MappedSuperClass`** : 이 클래스에 존재하는 필드도 컬럼으로 인식할 수 있도록 한다. 이 경우 `@MappedSuperClass` 어노테이션이 선언된 부모 클래스는 DB 테이블에 매핑되지 않는다. 즉 매핑될 컬럼만 상속받도록 해준다.

* **`@EntityListeners(AuditingEntityListener.class)`** : `BaseTimeEntity` 클래스에 Auditing 기능을 포함시킨다.

* **`@CreatedDate`** : Entity가 생성되면 저장될 때의 시간이 저장될 필드.

* **`@LastModifiedDate`** : Entity의 값이 변경될 때의 시간이 저장될 필드.

그리고 JPA Auditing 기능이 활성화 되도록 `Application` 클래스에 `@EnableJpaAuditing` 어노테이션을 추가시킨다.

```java
package com.jinsub.housekeeping;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@EnableJpaAuditing
@SpringBootApplication
public class HousekeepingBookApplication {

	public static void main(String[] args) {
		SpringApplication.run(HousekeepingBookApplication.class, args);
	}
}
```

## BaseTimeEntity 적용하기

앞서 생성한 `User`, `Category`, `Transaction` Entity 들을 모두 `BaseTimeEntity`를 상속하도록 변경한다. 그리고 Entity 내에 선언된 `createdAt`, `modifiedAt` 필드를 삭제한다. (테스트 코드에서 `createdAt`과 `modifiedAt`을 생성하거나 변경하는 코드도 모두 삭제합니다.)

```java
public class User extends BaseTimeEntity {
    ...
    
    /* createdAt, modifiedAt 삭제 */
}

public class Category extends BaseTimeEntity {
    ...
    
    /* createdAt, modifiedAt 삭제 */
}

public class Transaction extends BaseTimeEntity {
    ...
    
    /* createdAt, modifiedAt 삭제 */
}
```

마지막으로 위의 기능이 잘 동작하는지 테스트 코드를 작성한다. `User` Entity를 대상으로 테스트 코드를 작성해보기로 한다. `UserRepositoryTests` 클래스에 테스트 메소드를 다음과 같이 추가한다.

```java
@Test
public void 생성시간_수정시간_자동화() throws Exception {

    LocalDateTime now = LocalDateTime.of(2020, 6, 1, 0, 0, 0); // 2019년 6월 1일
    User savedUser = userRepository.save(User.builder()
            .userName("test user name")
            .hashedEmail(CryptoHelper.getSha256HashedString("test@email.com"))
            .hashedPassword(CryptoHelper.getSha256HashedString("test password"))
            .build());

    User testUser = userRepository.findById(savedUser.getUserId()).get();
    System.out.println("createdAt: " + testUser.getCreatedAt() + ", modifiedAt: " + testUser.getModifiedAt());

    assertThat(testUser.getCreatedAt()).isAfter(now);   // createdAt > now ?
    assertThat(testUser.getModifiedAt()).isAfter(now);  // modifiedAt > now ?
}
```
<br>

## Reference

> 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱, 프리렉, 2019