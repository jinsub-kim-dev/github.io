---
layout: post
title: "토이 프로젝트(3) : 데이터베이스 스키마 설계"
date: 2020-06-14 03:06:00 +0900
categories: Project
tags: Toy-project Spring-boot Spring MySQL Database
---

## 데이터베이스 스키마(Schema) 설계

프로젝트의 영구적 데이터를 관리하기 위해 DBMS로 `MySQL(version: 8.0.20)`을 사용한다. 지난 포스트에서 작성한 가계부의 초기 기능 명세를 참고하여 데이터베이스 스키마를 작성하였다. <br>
> 가계부 기능 명세 참고 : ["토이 프로젝트(1) : 구현할 기능에는 어떤 것들이 있을까?"]({% post_url  Project/2020-06-08-housekeeping-project-1 %})

## 테이블 설계

### tb_user
향후 로그인 기능을 추가하기 위해 이메일과 패스워드를 입력받는 필드를 추가하였다. 또한 이런 개인 정보를 DB에 직접 저장하고 있으면, 외부로부터 DB 데이터가 탈취됐을 때 정보가 그대로 유출된다. 따라서 이메일과 패스워드는 해싱된 값을 저장한다.

![image](/post_assets/2020-06-14/tb_user.png)

### tb_transaction
하나의 거래 내역 정보를 저장할 테이블이다. `transaction_type` 필드는 해당 내역이 지출인지 소비인지를 나타낸다. `asset_type` 필드는 현금 거래인지 또는 카드 거래인지를 나타낸다.

![image](/post_assets/2020-06-14/tb_transaction.png)

### tb_category
카테고리는 거래 출처 종류를 나타낸다. 예를 들어 월급, 식비, 교통, 편의점 등이 있다. 서비스를 사용하는 모든 사용자에게 공통으로 관리되는 카테고리와 사용자가 직접 추가할 수 있는 카테고리가 있다. 이를 분류하기 위해 `common_flag` 필드를 사용했다. 해당 값이 `true`면 모두 사용할 수 있는 공통 카테고리가 된다.

![image](/post_assets/2020-06-14/tb_category.png)

### tb_user_custom_category
사용자는 카테고리를 추가하거나 삭제할 수 있다. 이를 관리하기 위한 매핑 테이블이다. 만약 추가된 카테고리 이름이 중복인 경우, 기존에 저장된 카테고리를 그대로 사용한다. 때문에 `tb_category` 테이블의 `category_name` 필드에 유니크 키(Unique Key) 설정을 했다.

![image](/post_assets/2020-06-14/tb_user_custom_category.png)

### ERD(Entity-Relationship Diagram)

* 사용자(tb_user)는 여러 개의 거래 내역(tb_transaction)을 가질 수 있다.
* 한 개의 거래 내역(tb_transaction)에는 하나의 카테고리(tb_category)가 존재한다.
* 사용자(tb_user)는 여러 개의 커스텀 카테고리(tb_category)를 가질 수 있으며, 이름이 동일한 커스텀 카테고리는 여러 유저에 의해 사용될 수 있다.

![image](/post_assets/2020-06-14/erd.png)