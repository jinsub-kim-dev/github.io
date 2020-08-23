---
layout: post
title: "토이 프로젝트(10) : API 작성을 위한 DTO 설계"
date: 2020-08-23 11:59:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring
---
           
## DTO 설계

DTO(Data Transfer Object)는 문자 그대로 데이터 전송을 담당하는 객체이다. 여기서 "데이터 전송" 경로는 사용자가 호출한 API에서 서버의 비즈니스 로직까지를 말한다. 서비스 로직에서 데이터베이스 까지의 경로는 Entity 객체가 담당하게 된다. Entity는 데이터베이스 테이블을 표현하기 위해 정의되었기 때문에 데이터베이스 변경이 없는 한 속성을 추가하거나 삭제할 수 없다. 이러한 이유로 Entity가 존재함에도 DTO를 따로 정의해서 사용한다. 

DTO를 정의하고 활용하는 방법은 여러가지가 있지만, 여기선 하나의 대표 클래스안에 여러 내부 클래스를 둔다. 이렇게 설계한 이유는 내부 클래스를 둠으로써 해당 DTO가 어디에 속하는지 쉽게 알 수 있고, DTO의 종류가 많아지더라도 하나의 클래스안에서 관리하기 위함이다. `User`에 대한 생성 API를 작성하기 위해 다음과 같이 `UserDto`라는 이름의 클래스를 정의하였다.

```java
package com.jinsub.housekeeping.api.user.model.dto;

import com.jinsub.housekeeping.api.user.model.entity.User;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.io.Serializable;

public class UserDto implements Serializable {
    private static final long serialVersionUID = -6667329984959810070L;

    @Getter
    @Builder
    @NoArgsConstructor
    public static class CreateRequest {
        private String userName;
        private String email;
        private String password;
    }

    @Getter
    @Builder
    @NoArgsConstructor
    public static class CreateResponse {
        private long userId;
        private String userName;
        private String hashedEmail;
        private String hashedPassword;

        public static CreateResponse of(User user) {
            return CreateResponse.builder()
                    .userId(user.getUserId())
                    .userName(user.getUserName())
                    .hashedEmail(user.getHashedEmail())
                    .hashedPassword(user.getHashedPassword())
                    .build();
        }
    }
}
```

`UserDto`라는 클래스안에 `CreateRequest`와 `CreateResponse`라는 내부 클래스를 두었다. `CreateResponse` 클래스 내부에 정의된 static 메소드는 `User` Entity 객체를 DTO로 변환한다. 위에서 정의한 DTO를 사용하여 `User`를 새로 생성하는 서비스 로직과 API를 작성하였다.

```java
package com.jinsub.housekeeping.api.user.service;

import com.jinsub.housekeeping.api.user.model.entity.User;
import com.jinsub.housekeeping.api.user.repository.UserRepository;
import com.jinsub.housekeeping.base.exception.enums.HouseKeepingErrorType;
import com.jinsub.housekeeping.base.exception.model.HouseKeepingException;
import com.jinsub.housekeeping.base.helper.CryptoHelper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.security.NoSuchAlgorithmException;

@Service
public class UserService {

    // Repositories
    @Autowired
    UserRepository userRepository;

    public User createUser(String userName, String email, String password) throws NoSuchAlgorithmException {
        if (userRepository.existsByUserName(userName)) {
            throw new HouseKeepingException(HouseKeepingErrorType.POLICY_VIOLATION_ALREADY_EXIST_USER, "user is already exist");
        }

        return userRepository.save(User.builder()
                .userName(userName)
                .hashedEmail(CryptoHelper.getSha256HashedString(email))
                .hashedPassword(CryptoHelper.getSha256HashedString(password))
                .build());
    }
}
```

```java
package com.jinsub.housekeeping.api.controller;

import com.jinsub.housekeeping.api.user.model.dto.UserDto;
import com.jinsub.housekeeping.api.user.model.entity.User;
import com.jinsub.housekeeping.api.user.service.UserService;
import com.jinsub.housekeeping.base.model.CodeResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.security.NoSuchAlgorithmException;

@Controller
@RequestMapping("/api/v1/user")
public class UserController {

    @Autowired
    UserService userService;

    @PostMapping({"", "/"})
    @ResponseBody
    CodeResponse createUser(@RequestBody UserDto.CreateRequest request) throws NoSuchAlgorithmException {
        User createdUser = userService.createUser(request.getUserName(), request.getEmail(), request.getPassword());
        UserDto.CreateResponse createdUserResponse = UserDto.CreateResponse.of(createdUser);
        return CodeResponse.successResult(createdUserResponse);
    }
}
```