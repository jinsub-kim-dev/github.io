---
layout: post
title: "토이 프로젝트(7) : Transaction Entity와 연관 관계"
date: 2020-06-16 22:45:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring JPA
---

## JPA Entity의 연관 관계

데이터베이스는 외래키(FK; Foreign Key)를 사용하여 테이블 간 연관 관계를 설정할 수 있다. JPA에서의 연관 관계는 테이블이 아닌 객체(Object), 즉 Entity 간의 연관 관계를 의미한다. 그리고 외래키 대신 참조(Reference)를 사용하여 연관 관계를 설정한다. 코드를 작성하기 전에 먼저 JPA 연관 관계를 설명하기 위한 세 가지 개념을 간단히 정리한다.

### JPA 연관 관계 키워드 1 : 방향(Direction)

방향의 종류는 **단방향**과 **양방향**이 있다. 단방향은 서로 다른 두 Entity가 연관 관계가 있을 때 둘 중 한쪽만 참조할 수 있는 경우를 말한다. 간단한 예로 특정 팀(Team)에 소속된 선수(Player)를 생각해볼 수 있다.

```java
class Team {
    private long teamId;
}

class Player {
    private long playerId;
    private Team team;  // 이걸로 Team 객체에 접근할 수 있다!
}
```

`Player`는 멤버 변수 `team`을 통해 `Team` 클래스를 참조할 수 있다. 그러나 `Team` 클래스는 `Player`를 참조할 수 있는 방법이 없다. 만약 `Team`에도 `Player`를 참조할 수 있는 변수를 갖고 있다면 두 Entity의 연관 관계 방향은 양방향이 된다.

### JPA 연관 관계 키워드 2 : 다중성(Multiplicity)

위에서 사용한 예시로 계속 설명을 하자면 하나의 팀(Team)은 여러 선수(Player)를 보유할 수 있다. 그러나 한 선수(Player)는 반드시 하나의 팀(Team)에만 소속될 수 있다. 따라서 `Team`과 `Player`는 일대다(1:N) 관계다. 반대로 `Player`와 `Team`은 다대일(N:1) 관계다. 이 외에도 일대일(1:1), 다대다(N:M) 관계가 존재한다.

### JPA 연관 관계 키워드 3: 연관관계의 주인(owner)

데이터베이스에서 테이블은 두 테이블의 연관 관계를 설정하기 위해 **하나의 외래키**를 사용한다. JPA 에서 양방향 관계를 설정할 경우 각 Entity에 서로를 참조할 수 있는 변수를 선언하게 된다. 결국 외래키는 하나지만 두 개의 객체 참조가 존재하게 된다. 따라서 두 Entity 중 한쪽이 테이블의 외래키를 관리해야 하는데 이를 연관관계의 주인이라고 한다. 
> 자바 ORM 표준 JPA 프로그래밍 책의 일부를 참조하였습니다. 
<br>

## Transaction Entity와 User, Category Entity 사이의 연관 관계

먼저 이전 포스트에서 다룬 ERD(Entity-Relationship Diagram)를 다시 보면서 진행한다.

![image](/post_assets/2020-06-14/erd.png)

### Transaction과 User

먼저 `Transaction`과 `User`의 관계는 **다대일(N:1)**이다. 특정 거래(Transaction)에서 사용자를 참조할 수 있으며, 사용자는 자신의 모든 거래 내역을 조회할 수 있다. 따라서 **양방향 관계**를 갖도록 구현한다. 외래키는 `tb_transaction` 테이블에 존재하므로 **연관관계 주인은 `Transaction`**으로 결정한다.

### Transaction과 Category

다음으로 `Transaction`과 `Category`의 관계를 보자. 하나의 거래 내역(Transaction)은 한 종류의 카테고리(Category)를 가질 수 있다. 그러나 하나의 카테고리에 속한 거래 내역은 여러개가 존재할 수 있다. 따라서 `Transaction`과 `Category`는 **다대일(N:1)** 관계다. 향후 특정 카테고리 별 통계를 구하기 위해선 `Category`에서 `Transaction`을 참조할 수 있어야 한다. 따라서 **양방향 관계**를 갖도록 한다. `User`와 마찬가지로 **연관관계 주인은 `Transaction`이다.**

위의 연관 관계 설정 내용을 바탕으로 작성한 `Transaction`의 Entity는 다음과 같다.

```java
package com.jinsub.housekeeping.api.transaction.model.entity;

import com.jinsub.housekeeping.api.category.model.entity.Category;
import com.jinsub.housekeeping.api.transaction.enums.AssetType;
import com.jinsub.housekeeping.api.transaction.enums.TransactionType;
import com.jinsub.housekeeping.api.user.model.entity.User;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Transaction {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long transactionId;

    @ManyToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "user_id")
    private User user;

    @Column
    @Enumerated(EnumType.ORDINAL)
    private TransactionType transactionType;

    @Column
    private LocalDateTime transactionDate;

    @Column
    @Enumerated(EnumType.ORDINAL)
    private AssetType assetType;

    @ManyToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "category_id")
    private Category category;

    @Column
    private long amountOfMoney;

    @Column
    private String details;

    @Column
    private LocalDateTime createdAt;

    @Column
    private LocalDateTime modifiedAt;
}
```

`@ManyToOne` 어노테이션은 다대일 관계 표현에 사용되는 어노테이션이다. 여기서 사용된 속성인 `fetch`와 `cascade`에 관한 내용은 따로 포스팅할 예정이다.`@JoinColumn` 어노테이션은 외래 키 매핑에 사용된다. `name` 속성으로 매핑할 외래 키 이름(DB 컬럼명을 의미한다)을 지정할 수 있으며, 전달하지 않을 경우 **"필드명 + _ + 참조하는 테이블의 기본 키 컬럼명"**으로 만들어지는 값을 기본으로 사용한다.

양방향 연관 관계를 설정하기 위해 `User`와 `Category` Entity에 `Transaction`을 참조할 수 있는 변수를 다음과 같이 선언한다.
```java
public class User {
    ...
    @OneToMany(mappedBy = "user")
    private List<Transaction> transactionList = new ArrayList<>();
    ...
}
```
```java
public class Category {
    ...
    @OneToMany(mappedBy = "category")
    private List<Transaction> transactionList = new ArrayList<>();
    ...
}
```

`@OneToMany` 어노테이션으로 일대다(1:N) 관계를 설정한다. 여기서 `mappedBy` 속성은 양방향 관계일 경우에만 사용하며, 반대쪽 Entity 매핑의 필드 이름을 속성 값으로 전달하면 된다. `User`의 경우를 예로 들면 `Transaction` Entity에서 `User`를 참조하기 위한 필드가 있으며 필드명은 `user`이다(`Transaction.user` 이런식으로 참조). 따라서 `User`의 `@OneToMany` 어노테이션의 `mappedBy`의 속성 값은 `user`이다. 연관관계의 주인은 `mappedBy` 속성을 사용하지 않는다. 따라서 주인이 아닌 Entity가 해당 속성을 사용하여 연관관계의 주인이 누구인지 지정해야 한다.

> 연관관계 주인을 설정한다는 것은 외래 키를 관리할 대상을 지정하는 것다. 즉 외래키를 갖고 있는 테이블을 연관관계 주인으로 지정한다. 그런데 다대일 관계에서는 외래키는 항상 "다(Many)"쪽에서 관리하게 된다. 따라서 "다(Many)"에 해당하는 `@ManyToOne` 어노테이션은 `mappedBy` 속성이 존재하지 않는다.

## Transaction Repository 생성

`Transaction`의 Repository 코드는 다음과 같다. (테스트 코드는 생략한다. 대신 [Github](https://github.com/jinsub-kim-dev/toy-project-housekeeping-book)에서 확인할 수 있다.)

```java
package com.jinsub.housekeeping.api.transaction.repository;

import com.jinsub.housekeeping.api.transaction.model.entity.Transaction;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TransactionRepository extends JpaRepository<Transaction, Long> {
}
```

<br>

## Reference

> 자바 ORM 표준 JPA 프로그래밍, 김영한, 에이콘, 2015