---
layout: post
title: "토이 프로젝트(5) : User에 대한 Entity, Repository 생성 그리고 Test"
date: 2020-06-14 22:46:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring JPA
---

## User Entity 생성

데이터베이스의 `tb_user` 테이블에 해당하는 자바 객체, User Entity를 생성해본다. Entity를 생성할 패키지 경로는 `com.jinsub.housekeeping.api.user.model.entity` 이다. 앞으로 클라이언트에 제공하는 API와 직접적으로 관련된 모든 코드는 `api`에 작성한다. (특정 API에 속하지 않고 범용적으로 사용되는 코드는 `com.jinsub.housekeeping.base` 경로에 작성한다.)

```java
package com.jinsub.housekeeping.api.user.model.entity;

import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity(name = "tb_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long userId;

    @Column
    private String userName;

    @Column
    private String hashedEmail;

    @Column
    private String hashedPassword;

    @Column
    private LocalDateTime createdAt;

    @Column
    private LocalDateTime modifiedAt;
}

```
`@Entity` 어노테이션을 통해 해당 클래스가 JPA에 의해 관리됨을 나타낸다. `name` 속성에 해당하는 테이블에 매핑되며, 없을 시 클래스 명의 Snake Case에 해당하는 이름으로 자동 매핑된다. `@Id`는 매핑될 테이블의 PK를 나타낸다. `@GeneratedValue` 어노테이션과 `GenerationType.IDENTITY`를 통해 Auto increment 임을 나타내었다. `@Column` 어노테이션을 사용하여 필드와 테이블의 컬럼을 매핑한다. 굳이 선언하지 않아도 Entity의 모든 필드는 테이블 컬럼과 자동으로 매핑된다.

## User Repository 생성

`tb_user` 테이블에 접근하기 위한 DB Layer 객체이다. MyBatis에서 DAO(Data Access Object)에 해당한다. 인터페이스로 생성한 후, `JpaRepository<Entity 클래스, PK 타입>`를 상속받으면 해당 Entity에 대한 기본적인 CRUD 메소드가 자동으로 생성된다. 패키지 경로 `com.jinsub.housekeeping.api.user.repository`에 `UserRepository`를 생성한다.

```java
package com.jinsub.housekeeping.api.user.repository;

import com.jinsub.housekeeping.api.user.model.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

## User Repository 테스트

프로젝트 경로 `src/main/test`에 UserRepository와 동일한 패키지 경로를 생성한 후 `UserRepositoryTests`라는 이름의 클래스를 생성한다. 그리고 아래와 같이 User Entity를 저장하고 불러오는 테스트 코드를 작성한다.

```java
package com.jinsub.housekeeping.api.user.repository;


import com.jinsub.housekeeping.api.user.model.entity.User;
import com.jinsub.housekeeping.base.helper.CryptoHelper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
// @AutoConfigureTestDatabase(replace = Replace.NONE)
public class UserRepositoryTests {

    @Autowired
    UserRepository userRepository;

    @Test
    public void 유저를_생성한다() throws Exception {

        String testUserName = "test user name";
        String testUserEmail = "test@email.com";
        String testUserPassword = "test_user_password";

        User savedUser = userRepository.save(User.builder()
                .userName(testUserName)
                .hashedEmail(CryptoHelper.getSha256HashedString(testUserEmail))
                .hashedPassword(CryptoHelper.getSha256HashedString(testUserPassword))
                .createdAt(LocalDateTime.now())
                .modifiedAt(LocalDateTime.now())
                .build());

        User testUser = userRepository.findById(savedUser.getUserId()).get();

        assertThat(testUser.getUserName()).isEqualTo(testUserName);
        assertThat(testUser.getHashedEmail()).isEqualTo(CryptoHelper.getSha256HashedString(testUserEmail));
        assertThat(testUser.getHashedPassword()).isEqualTo(CryptoHelper.getSha256HashedString(testUserPassword));
    }
}
```
> 코드 중에 `CryptoHelper.getSha256HashedString()` 메소드가 있다. 이 메소드는 User의 이메일과 패스워드를 해싱(Hashing)하는 기능을 수행한다. 특정 API에 속하지 않는 기능이라 판단하여 `base` 패키지 경로에 코드를 작성하였다. 해당 코드는 [Github](https://github.com/jinsub-kim-dev/toy-project-housekeeping-book)에서 확인할 수 있다.

`@DataJpaTest` 어노테이션을 사용하면 JPA 기능을 테스트 할 수 있게 된다. 그러나 해당 어노테이션을 사용하려면 인메모리 데이터베이스가 필요하다. 그래서 JPA 설정 과정에서 H2 데이터베이스를 추가로 설정한 것이다. 만약 실제 데이터베이스로 테스트를 진행하고 싶으면 `@AutoConfigureTestDatabase(replace = Replace.NONE)` 어노테이션을 추가하면 된다. 이 경우 인메모리가 아닌 실제 데이터베이스를 대상으로 테스트를 진행하게 된다. 

인텔리제이 화면에서 테스트 메소드 옆에 테스트 코드를 실행할 수 있는 녹색 버튼이 있을 것이다. 그곳에서 `Run 유저를_생성한다`를 클릭하면 해당 테스트 메소드를 실행하게 된다. 테스트 클래스 선언부에도 동일한 UI가 존재하는데 클릭하면 해당 클래스 내의 모든 테스트 메소드를 실행할 수 있다.

![image](/post_assets/2020-06-14/user_repository_test.png)
<br>


## Reference

> 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱, 프리렉, 2019 <br>
> 자바 ORM 표준 JPA 프로그래밍, 김영한, 에이콘, 2015