---
layout: post
title: "토이 프로젝트(2) : 프로젝트 생성"
date: 2020-06-13 14:53:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring
---

## 프로젝트 생성

프로젝트는 스프링 부트를 통해 구현해 나갈 것이다. 프로젝트 생성 및 초기화는 [Spring Initializr](https://start.spring.io/) 에서 진행한다. 아래와 같이 필요한
정보를 입력하고 **GENERATE** 버튼을 클릭하여 프로젝트를 생성한다. (`"우클릭 > 새 탭에서 이미지 열기"`로 큰 이미지를 볼 수 있습니다.)

![image](/post_assets/2020-06-13/spring-initializer.png)

### 프로젝트 생성 정보

- Project : 스프링 의존성 관리 및 빌드에 사용될 툴. **Maven Project**를 선택.
- Language : 프로젝트 프로그래밍 언어. **Java**를 선택.
- Spring Boot : 스프링 부트 버전을 지정. **2.3.1**을 선택.
- Project Metadata :
  - Group : 패키지 구성에 사용되는 경로, 도메인의 역순으로 쓴다.
  - Artifact : 프로젝트 공식 이름.
  - Package Name : 프로젝트 패키지 경로.
  - Packaging : 패키징 방식. **Jar**를 선택.
  - Java : 사용할 Java 버전. **8**을 선택.

- Dependency
  - MySQL, Lombok, JPA 등 여러 의존성을 추가할 수 있다.
  - 여기선 **Spring Web** (`spring-boot-starter-web`) 의존성만 추가한다.


## 생성된 프로젝트 확인
인텔리제이(IntelliJ) 실행 > `Open or import`를 통해 생성 프로젝트를 열면 이제 코드를 작성할 수 있는 개발 환경을 구축할 수 있다. 프로젝트 루트 디렉토리에서 `pom.xml` 파일을 열면 Spring Initializr에서 설정한 내용이 반영되어 있음을 확인할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.15.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.jinsub.housekeeping</groupId>
	<artifactId>housekeeping-book-spring-boot-project</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>housekeeping-book</name>
	<description>Toy project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```