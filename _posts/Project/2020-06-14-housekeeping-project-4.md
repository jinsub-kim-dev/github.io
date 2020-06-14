---
layout: post
title: "토이 프로젝트(4) : MySQL 연동과 JPA 사용"
date: 2020-06-14 12:28:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring Maven
---

## MySQL 연동하기

`pom.xml` 파일을 열어서 다음과 같이 MySQL을 사용하기 위한 의존성 정보를 입력한다.

```xml
<dependencies>
    ...
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    ...
</dependencies>
```

그리고 `src/main/resources/application.properties` 파일을 열어 다음과 같이 Connection 정보를 입력한다.<br>
(현재는 로컬 DB를 사용하고 있지만, 향후 DB 서버를 따로 구축하고 Replication을 통한 로드 밸런싱을 진행한다.)
```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/housekeeping?serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.username=jinsub
spring.datasource.password=jinsub
```


## JPA(Java Persistence API) 사용하기

서버 개발 시 데이터를 영구적으로 저장하기 위한 목적으로 반드시 데이터베이스를 사용하게 된다. 주로 관계형 데이터베이스(RDB)가 사용되며, SQL(Structed Quert Language)을 통해 데이터의 CRUD(Create, Read, Update, Delete)가 이루어진다. 하지만 스프링 프레임워크는 **객체(Object)**가 중심이 되는 언어인 자바를 사용하기 때문에 RDB와 근본적인 형태가 다르다고 할 수 있다. 이 차이를 메꾸기 위해 JDBC, MyBatis 등의 기술이 사용되었다. 그러나 프로젝트 규모가 커지게 될 수록 SQL 작성 및 객체를 DB 테이블에 매핑하기 위한 과정에서 유지/보수 난이도가 상승하는 문제점이 발생하였다. ORM(Object-Relational Mapping) 프레임워크는 이러한 문제를 해결하기 위해 등장했고, JPA는 자바 진영의 ORM 기술 표준으로 사용되고 있다. 본 프로젝트에선 RDB를 사용하기 위해 JPA를 사용할 것이다.

`pom.xml` 파일을 열어서 스프링 부트에서 JPA를 사용하기 위한 의존성 정보를 입력하면 간단히 JPA 설정이 완료된다.

```xml
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    ...
</dependencies>
```