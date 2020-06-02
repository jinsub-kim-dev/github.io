---
layout: post
title:  "스트림(Stream) (2) : 스트림의 연산"
date:   2020-02-20 01:04:32 +0900
categories: Java
tags: Java
---

## 1. 중간 연산
스트림이 배열을 대상으로 생성되었건 컬렉션 인스턴스를 대상으로 생성되었건, 이에 상관없이 동일한 방법으로 중간 연산 및 최종 연산을 진행할 수 있다. (적용 가능한 연산 종류에 약간의 차이가 존재)

### 1-1. 필터링(Filtering)
**필터링**은 스트림 내 요소들을 하나씩 평가해서 걸러내는 작업을 수행한다. 필터링에 사용되는 메소드는 다음과 같다.
```java
Stream<T> filter(Predicate<? super T> predicate)
```
이 때 매개변수로 `Predicate`의 다음 추상 메소드의 구현에 해당하는 람다식을 인자로 전달해야한다.
```java
Predicate<T> boolean test(T t)
```

다음 Java 코드는 정수 배열과 문자열 배열에서 특정 조건에 해당하는 요소를 출력하는 코드이다.
```java
int[] ar = {1, 2, 3, 4, 5};
Arrays.stream(ar)  // 배열 기반 스트림 생성
      .filter(n -> n % 2 == 1)  // 홀수만 통과시킨다.
      .forEach(n -> System.out.print(n + "\t"));  // 1    3    5

List<String> sl = Arrays.asList("Toy", "Robor", "Box");
sl.stream()  // 컬렉션 인스턴스 기반 스트림 생성
    .filter(s -> s.length() == 3)  // 길이가 3이면 통과시킨다.
    .forEach(s -> System.out.print(s + "\t"));  // Toy  Box
```

### 1-2. 맵핑(Mapping)
맵(_map_)은 스트림 내 요소들을 **특정한 맵핑 기준**에 따라 변환해주는 **중간 연산**이다. 다음은 문자열 배열을 대상으로 스트림을 생성하고 "문자열의 길이"라는 기준으로 맵핑을 적용하는 코드이다.
```java
List<String> ls = Arrays.asList("Box", "Robot", "Simple");
ls.stream()
    .map(s -> s.length())
    .forEach(n -> System.out.print(n + "\t"));  // 3    5    6
```

맵핑에 사용되는 대표적인 메소드는 다음과 같다.
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```
매개변수가 `Function`이므로 다음 메소드의 구현에 해당하는 람다식을 인자로 전달해야 한다.
```java
Function<T, R> R apply(T t)
```
위의 메소드는 제네릭 타입 형태이기 때문에 정수의 반환 과정에서 오토 박싱이 진행된다. 그래서 Java에서는 기본 자료형의 값을 반환하는 경우를 고려하여 다음의 맵핑 관련 메소드들도 제공하고 있다.
```java
IntStream mapToInt(ToIntFunction<? super T> mapper)
LongStream mapToLong(ToLongFunction<? super T> mapper)
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)
```

### 1-3. 정렬
정렬 기능을 제공하는 중간 연산 메소드들은 다음과 같다.
```java
Stream<T> sorted(Comparator<? super T> comparator)
Stream<T> sorted(Comparator<? super T> comparator)
IntStream sorted(Comparator<? super T> comparator)
LongStream sorted(Comparator<? super T> comparator)
DoubleStream sorted(Comparator<? super T> comparator)
```
이 중 첫 번째 메소드를 사용하기 위해선 `Comparator<T>` 인터페이스의 `compare`메소드 구현에 해당하는 람다식을 전달해야 한다.
```java
Stream.of("Rabbit", "Box", "Apple")
      .sorted((s1, s2) -> s1.length() - s2.length())
      .forEach(s -> System.out.print(s + "\t"));  // Box    Apple    Rabbit
```

## 2. 최종 연산
가공한 스트림을 바탕으로 결과값을 만들어내는 단계이다. 즉 스트림을 끝내는 최종 작업에 해당한다.

### 2-1. Calculating
최소, 최대, 합, 평균 등의 결과를 계산할 수 있다. (스트림이 비어있는 경우 `count`와 `sum`은 0을 출력하지만 평균, 최소, 최대의 경우에는 표현할 수 없기 때문에 `Optional`을 이용해 리턴한다.)
```java
long count = IntStream.of(1, 3, 5, 7, 9).count();  // 5
long sum = LongStream.of(1, 3, 5, 7, 9).sum();  // 25
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min(); // 1
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max(); // 9
```

### 2-2 리덕션(Reduction) & 병렬 스트림(Parallel Streams)
### 2-2-1. 리덕션에 사용되는 reduce 메소드
리덕션(Reduction)은 "데이터를 축소하는 연산"을 뜻한다. `sum`도 리덕션 연산에 해당한다. 또한 리덕션을 수행하기 위한 기준을 사용자가 정의할 수도 있으며 이 때 사용하는 메소드는 다음과 같다.
```java
T reduce(T identity, BinaryOperator<T> accumulator)
```
`reduce` 메소드의 두 번째 인자로 다음의 추상 메소드를 람다식으로 구현하여 전달한다.
```java
BinaryOperator<T>    T apply(T t1, T t2)
```
`reduce`를 호출하면 내부적으로 `apply`를 호출하면서 스트림에 저장된 데이터를 다음과 같이 줄여나간다.

![image](/post_assets/2020-02-20/reduce.png)
> Image 참고 : 윤성우의 열혈 Java 프로그래밍, 윤성우, 오렌지미디어, 2017, p.747

문자열 길이를 기준으로 리덕션을 수행하는 예
```java
List<String> ls = Arrays.asList("Box", "Simple", "Complex", "Robot");

BinaryOperator<String> lc = (s1, s2) -> {
    if(s1.length() > s2.length())
        return s1;
    else
        return s2;
};

String str = ls.stream().reduce("", lc);
System.out.println(str);  // Complex
```

### 2-2-2. 병렬 스트림(Parallel Streams)
하나의 작업을 둘 이상의 작업으로 나누어 동시에 진행하는 것을 가리켜 "병렬 처리"라고 한다. Java는 언어 차원에서 병렬 처리를 지원하고 있다. 위의 예제를 "병렬 처리" 기반으로 수정한 코드는 다음과 같다.
```java
// String str = ls.stream().reduce("", lc);
String str = ls.parallelStream().reduce("", lc);
```
병렬 스트림을 생성하면 이어지는 연산들은 CPU의 코어 개수를 고려하여 적절하게 병렬 처리가 된다. 

  
![image](/post_assets/2020-02-20/parallel-processing.png)
> Image 참고 : 윤성우의 열혈 Java 프로그래밍, 윤성우, 오렌지미디어, 2017, p.751


### 2-3. Matching

`Stream<T>` 인터페이스에는 다음 메소드들이 존재한다.
```java
// 스트림의 데이터가 조건을 모두 만족하는가?
boolean allMatch(Predicate<? super T> predicate)

// 스트림의 데이터가 조건을 하나라도 만족하는가?
boolean anyMatch(Predicate<? super T> predicate)

// 스트림의 데이터가 조건을 하나도 만족하지 않는가?
boolean noneMatch(Predicate<? super T> predicate)
```
또한 `IntStream`, `DoubleStream`, `LongStream` 인터페이스에도 위와 동일한 메소드들이 존재한다.

## Reference
> https://futurecreator.github.io/2018/08/26/java-8-streams/ <br>
> https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/ <br>
> 윤성우의 열혈 Java 프로그래밍, 윤성우, 오렌지미디어, 2017