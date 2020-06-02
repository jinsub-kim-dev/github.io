---
layout: post
title:  "스트림(Stream) (1) : 스트림의 개요 및 생성"
date:   2020-02-20 00:52:49 +0900
categories: Java
tags: Java
---

## 1. 스트림 개요
Java 8에서 추가된 스트림은 람다를 활용할 수 있는 기술이다. Java 8 이전의 배열 또는 컬렉션 인스턴스를 다루는 방법은 `for` 또는 `foreach` 문을 사용하여 요소 하나씩 꺼내서 다루는 것이었다. 로직이 복잡해질수록 여러 로직이 섞이게 되고 분기 발생 시 여러 번의 루프를 도는 경우가 발생하기도 한다.

내부가 비어있는 파이프에 물을 흘려보내면 다른 쪽 입구를 통해 물이 흘러나온다. 이와 유사하게 Java에서도 **데이터의 흐름**을 생성할 수 있으며, 이러한 데이터의 흐름을 가리켜 "스트림(Stream)"이라고 한다. 물이 흐르는 파이프는 물을 흘려보내는 공간을 제공할 뿐만 아니라, 정화, 수압 조절 등의 기능을 제공하기도 한다. **마찬가지로 Java의 스트림은 배열 또는 컬렉션 인스턴스를 대상으로 여러 연산을 수행하는 파이프를 생성할 수 있으며, 파이프의 출력을 또 다른 파이프의 입력으로 사용할 수 있다.**

스트림에 대한 내용은 크게 세 가지로 나눌 수 있다.
1. 스트림 생성 : 스트림 인스턴스 생성
2. 중간 연산 : 마지막이 아닌 위치에서 진행되는 연산. 필터링(filtering), 맵핑(mapping) 등
3. 최종 연산 : 마지막에 진행되어야 하는 연산. 최종적인 결과물을 만들어내는 작업.

## 2. 스트림 생성
스트림 생성은 배열 및 컬렉션 인스턴스를 대상으로 스트림을 생성하는 방법을 말한다.

### 2-1. 스트림 생성 : 배열
배열에 저장된 데이터를 대상으로 스트림을 생성할 때 호출하는 대표 메소드.
```java
public static <T> Stream<T> stream(T[] array); // Arrays 클래스에 정의
```
```java
String[] names = {"Kim", "Jin", "Sub"};
Stream<String> stm = Arrays.stream(names);  // 스트림 생성
stm.forEach(s -> System.out.println(s));    // 최종 연산 진행
```
```java
String[] names = {"Kim", "Jin", "Sub"};
Arrays.stream(names)
      .forEach(s -> System.out.println(s));
```
기본(Primitive) 자료형의 값을 저장하는 배열을 대상으로 스트림을 생성하는 방법들
```java
public static IntStream stream(int[] array)
public static IntStream stream(int[] array, int startInclusive, int endExclusive)
public static DoubleStream stream(double[] array)
public static DoubleStream stream(double[] array, int startInclusive, int endExclusive)
public static LongStream stream(long[] array)
public static LongStream stream(long[] array, int startInclusive, int endExclusive)
```

### 2-2. 스트림 생성 : 컬렉션 인스턴스
컬렉션 인스턴스를 대상으로 스트림을 생성하는 `stream`메소드는 `java.util.Collection<E>`에 디폴트 메소드로 정의되어 있다.
```java
default Stream<E> stream()
```
```java
List<String> list = Arrays.asList("Toy", "Robot", "Box");
list.stream()
    .forEach(s -> System.out.print(s + "\t"));
```

### 2-3. 다양한 스트림 생성
**비어 있는 스트림**
```java
Stream<String> stream = Stream.empty();
```
**_Stream.builder()_**
```java
Stream<String> buliderStream = 
    Stream.<String>builder()
          .add("Toy").add("Robot").add("Box")
          .build();  // [Toy, Robot, Box]
```
**_Stream.generate()_**
```java
Stream<String> buliderStream = 
    Stream.generate(() -> "Hello").limit(5);  // [Hello, Hello, Hello, Hello, Hello]
```
**_Stream.iterate()_**
```java
Stream<Integer> iteratedStream = 
    Stream.iterate(30, n -> n + 2).limit(5);  // [30, 32, 34, 36, 38]
```
**문자열 스트림**
```java
IntStream charsStream = "Stream".chars();  // [83, 116, 114, 101, 97, 109]
```
**스트림 생성에 필요한 데이터를 직접 전달**
`Stream<T>` 인터페이스에는 데이터를 인자로 직접 전달함으로써 스트림을 생성하는 메소드가 존재한다.
```java
static <T> Stream<T> of(T t)
static <T> Stream<T> of(T...values)
```

또한 인터페이스 `Stream<T>`의 타입 매개변수 `T`에 `int`와 같은 기본 자료형의 이름이 올 수 없으므로 Java에서는 다음의 메소드들도 제공하고 있다.
```java
/* DoubleStream 인터페이스의 메소드 */
static DoubleStream of(double...values)
static DoubleStream of(double t)

/* IntStream 인터페이스의 메소드 */
static IntStream of(int...values)
static IntStream of(int t)

/* LongStream 인터페이스의 메소드 */
static LongStream of(long...values)
static LongStream of(long t)
```
해당 메소드를 통해 기본 자료형 데이터로 이루어진 스트림을 생성하면, 불필요한 오토 박싱/언박싱을 피할 수 있다.
그리고 다음 메소드들을 통해 범위 내의 값들로 스트림을 생성할 수 있다.
```java
/* IntStream 인터페이스의 메소드 */
static IntStream range(int startInclusive, int endExclusive)  // start <= n < end
static IntStream rangeClosed(int startInclusive, int endExclusive) // start <= n <= end

/* LongStream 인터페이스의 메소드 */
static LongStream range(long startInclusive, long endExclusive)  // start <= n < end
static LongStream rangeClosed(long startInclusive, long endExclusive) // start <= n <= end
```

**스트림의 연결**
두 개의 스트림을 연결하여 하나의 스트림을 생성할 수도 있다.
```java
static <T> Stream<T> concat(Stream<?  extends T> a, Stream<? extends T> b)
static DoubleStream concat(DoubleStrema a, DoubleStream b)
static IntStream concat(IntStream a, IntStream b)
static LongStream concat(LongStream a, LongStream b)
```
```java
Stream<String> ss1 = Stream.of("Cake", "Milk");
Stream<String> ss2 = Stream.of("Lemon", "Jelly");

Stream.concat(ss1, ss2).forEach(s -> System.out.println(s)); // Cake Milk Lemon Jelly
```

## Reference
> https://futurecreator.github.io/2018/08/26/java-8-streams/ <br>
> https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/ <br>
> 윤성우의 열혈 Java 프로그래밍, 윤성우, 오렌지미디어, 2017