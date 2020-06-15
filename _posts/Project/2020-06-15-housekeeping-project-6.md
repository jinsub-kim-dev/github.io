---
layout: post
title: "토이 프로젝트(6) : Category에 대한 Entity, Repository 생성 그리고 Test"
date: 2020-06-15 22:45:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring JPA
---

## Category Entity 생성

User와 마찬가지로 Category 관련 코드는 `category`라는 이름의 패키지 경로를 생성하고 그곳에 작성한다. 먼저 Category Entity를 `com.jinsub.housekeeping.api.category.model.entity` 경로에 생성한다.

```java
package com.jinsub.housekeeping.api.category.model.entity;

import com.jinsub.housekeeping.api.transaction.enums.TransactionType;
import com.jinsub.housekeeping.base.converter.BooleanToYNConverter;
import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity(name = "tb_category")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long categoryId;

    @Column
    private String categoryName;

    @Column
    @Enumerated(EnumType.ORDINAL)
    private TransactionType transactionType; // INCOME(0), EXPENSE(1)

    @Column(name = "common_flag")
    @Convert(converter = BooleanToYNConverter.class)
    private boolean common;

    @Column
    private LocalDateTime createdAt;

    @Column
    private LocalDateTime modifiedAt;
}
```
`TransactionType` 타입은 수입과 지출을 표현하는 Enumeration이다. `@Enumerated(EnumType.ORDINAL)` 어노테이션은 Enum에 정의된 순서대로 `INCOME`은 0, `EXPENSE`는 1 값이 데이터베이스에 저장되도록 한다.

`common` 필드에 해당하는 데이터베이스 컬럼명은 `common_flag`이다. `@Column(name = "common_flag")` 어노테이션을 사용하여 필드가 컬럼에 올바르게 매핑되도록 한다. 이름 말고 다른점이 또 있는데, `common` 필드는 `boolean` 타입이지만 데이터베이스 컬럼은 `CHAR(1)` 타입이다. 따라서 데이터베이스 조회, 입력 과정에서 타입 불일치를 해결해줘야한다. 이를 위해 사용되는 것이 `@Convert` 어노테이션이다. converter에 등록될 객체는 `AttributeConverter<X, Y>` 인터페이스를 구현해야 한다.

```java
package com.jinsub.housekeeping.base.converter;

import javax.persistence.AttributeConverter;

public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {

    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        // Entity의 데이터를 데이터베이스 컬럼에 저장할 데이터로 변환한다.
        return (attribute != null && attribute.equals(Boolean.TRUE)) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        // 데이터베이스에서 조회한 컬럼 데이터를 Entiry의 데이터로 변환한다.
        return dbData.equals("Y");
    }
}
```

## Category Repository 생성

패키지 경로 `com.jinsub.housekeeping.api.category.repository`에 `CategoryRepository`를 생성한다.

```java
package com.jinsub.housekeeping.api.category.repository;

import com.jinsub.housekeeping.api.category.model.entity.Category;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CategoryRepository extends JpaRepository<Category, Long> {
}
```

## Category Repository 테스트

Category Entity를 생성하고 제대로 저장되었는지 조회하는 테스트 코드를 작성한다. 테스트 대상과 동일한 경로에 테스트 코드를 생성한다. `src/test/java/com/jinsub/housekeeping/api/category/repository/`에 `CategoryRepositoryTests`라는 이름의 클래스를 생성한다.

```java
package com.jinsub.housekeeping.api.category.repository;

import com.jinsub.housekeeping.api.category.model.entity.Category;
import com.jinsub.housekeeping.api.transaction.enums.TransactionType;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
public class CategoryRepositoryTests {

    @Autowired
    CategoryRepository categoryRepository;

    @Test
    public void 카테고리를_생성한다() {

        String testCategoryName = "test category name";
        TransactionType testTransactionType = TransactionType.EXPENSE;
        boolean testCommon = true;

        Category savedCategory = categoryRepository.save(Category.builder()
                .categoryName(testCategoryName)
                .transactionType(testTransactionType)
                .common(testCommon)
                .createdAt(LocalDateTime.now())
                .modifiedAt(LocalDateTime.now())
                .build());

        Category testCategory = categoryRepository.findById(savedCategory.getCategoryId()).get();

        assertThat(testCategory.getCategoryName()).isEqualTo(testCategoryName);
        assertThat(testCategory.getTransactionType()).isEqualTo(testTransactionType);
        assertThat(testCategory.isCommon()).isEqualTo(testCommon);
    }
}
```

이제 남은건 Transaction에 대한 Entity, Repository 작성이다. 여기서부턴 테이블 관계에 따른 JPA의 연관 관계 설정에 대한 내용이 추가될 것이다. 
<br>

## Reference

> 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱, 프리렉, 2019 <br>
> 자바 ORM 표준 JPA 프로그래밍, 김영한, 에이콘, 2015