---
layout: post
title:  "자바 8의 람다 표현식(Lambda Expression)"
date:   2020-02-19 00:08:37 +0900
categories: Java
tags: Java
---

## 1. 람다식 개요
람다 표현식(Lambda Expression)은 `기능 하나를 정의해서 전달해야 하는 상황`에 사용될 수 있다.  `Comparator<T>` 인터페이스의 구현이 필요한 상황이 대표적인 예이다.
```java
class SLenComp implements Comparator<String> {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}

class SLenComparator {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Robot");
        list.add("Box");
        list.add("Lambda");

        Collections.sort(list, new SLenComp());

        for(String s : list) 
            System.out.println(s);
    }
}
```

위의 예제는 "리스트의 정렬"이라는 기능을 전달하기 위해 `Comparator<T>` 인터페이스를 구현한 `SLenComp` 클래스를 정의한 것이다. 여기에 람다 표현식을 적용하면 리스트를 정렬하는 코드를 다음과 같이 작성할 수 있다.
```java
Collections.sort(list, (s1, s2) -> s1.length() - s2.length());
```

또 하나의 대표적인 예는 Thread를 생성하는 것이다.
```java
/* Thread - traditional */
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World.");
    }
}).start();

/* Thread - Lambda Expression */
new Thread(()->{
    System.out.println("Hello World.");
}).start();
```
위의 예제를 통해 확인할 수 있는 람다 표현식의 장점은 불필요한 코드를 줄이고 가독성을 높인다는 것이다.

## 2. 람다식 표현 방법
* 람다식의 좌변에는 메소드의 매개변수를, 우변에는 메소드의 몸체를 선언한다. 
* 메소드 몸체가 둘 이상의 문장으로 이뤄져 있거나, 매개변수가 둘 이상인 경우에는 각각 중괄호와 소괄호의 생략이 불가능하다. 
* 중괄호를 생략할 경우 해당 문장의 끝에 위치한 세미콜론도 함께 지워야 한다. 
* 메소드 몸체에 해당하는 내용이 `return`문이라면, 그 문장이 하나이더라도 중괄호의 생략이 불가능하다.

```java
( parameters ) -> expression body
( parameters ) -> { expression body }
( ) -> expression body
( ) -> { expression body }
```

## 3. 함수형 인터페이스 및 @FunctionalInterface 어노테이션
추상 메소드가 딱 하나만 존재하는 인터페이스를 `함수형 인터페이스(Functional Interface)`라고 한다. 그리고 람다식은 함수형 인터페이스를 기반으로만 작성이 될 수 있다.
```java
@FunctionalInterface
interface Calculate {
    int cal(int a, int b);
}
```

`@FunctionalInterface`는 함수형 인터페이스에 부합하는지 확인하기 위한 어노테이션이다. 해당 어노테이션을 갖는 인터페이스가 둘 이상의 추상 메소드를 갖는 경우 컴파일 오류로 이어진다. 그러나 `static`, `default` 선언이 붙은 메소드는 함수형 인터페이스의 정의에 아무런 영향을 미치지 않는다. 따라서 다음의 인터페이스도 함수형 인터페이스에 부합한다.
```java
@FunctionalInterface
interface Calculate {
    int cal(int a, int b);
    default int add(int a, int b) { return a + b; }
    static int sub(int a, int b) { return a - b; }
}
```

## Reference
> https://jdm.kr/blog/181 <br>
> https://multifrontgarden.tistory.com/124 <br>
> https://multifrontgarden.tistory.com/125?category=471239 <br>
> 윤성우의 열혈 Java 프로그래밍, 윤성우, 오렌지미디어, 2017 