---
layout: post
title:  "롬복(Lombok) 플러그인 설치"
date:   2020-06-14 18:20:00 +0900
categories: Spring
tags: Spring Lombok
---

## 롬복(Lombok)이란?

개발을 하다 보면 동일한 기능을 수행하지만 꼭 필요한 코드이기 때문에 반복적으로 작성해야 하는 것들이 있다. 예를 들면 클래스의 생성자, Getter/Setter 등. 롬복(Lombok)은 이러한 코드를 어노테이션을 통해 자동으로 생성해주는 플러그인이다. 작업 속도를 굉장히 빠르게 해주는 편리한 기능을 갖고 있기 때문에 대부분의 자바 프로젝트에서 사용한다. 

## 인텔리제이(InjelliJ)에 롬복 설치하기

먼저 라이브러리 의존성 설정을 해준다. Maven을 기준으로 `pom.xml` 파일에 다음을 추가한다.
```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```

인텔리제이의 설정(Windows의 경우 File > Settings...)에서 왼쪽 검색 창에 "plugins"라고 입력한다. 그러면 플러그인을 설치할 수 있는 화면으로 변경되고 여기에 "Lombok"을 입력하여 플러그인을 설치한다. 

![image](/post_assets/2020-06-14/lombok-install.png)

플러그인이 설치가 완료되면 자동으로 인텔리제이가 재시작 하게 된다. File > Settings > Build, Execution, Deployment > Compiler > Annotation Processors 로 접근하여 Enable annotation processing 항목을 체크한 후 저장하면 롬복의 기능을 사용할 수 있다.

![image](/post_assets/2020-06-14/lombok-install2.png)