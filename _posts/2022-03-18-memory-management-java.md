---
layout: post
title: 메모리 관리(Memory Management in Java)
tags: CS, memory-management
math: true
date:  2022-03-18 16:35 
---

# 프로세스
정의 : 실행중인 프로그램 ( A program in execution)
![](https://images.velog.io/images/dlrlejr132/post/1d0da571-fbba-4c8c-9ac7-9ab79274b51e/image.png)

# 프로세스 메모리 구조

#### 프로세스 메모리 구조는 Text, Data, Heap, Stack 영역으로 구분 되어 있다.

**Text** : 프로그램 코드와 상수가 정의되어 있고, 읽기만 가능한 메모리 영역이기 때문에 데이터를 저장하려고 하면 분할 충돌을 일으켜 프로세스가 중지된다.
**Data** : 전역 변수(Global variable)와 정적 변수(Static variable)가 저장되어 있는 영역.
**Heap**: 프로그래머의 필요에 따라 동적 메모리 호출에 의해 할당되는 메모리 영역. C언어의 기준으로 malloc() 함수나 calloc() 함수에 의해 생성된 변수들이 이 곳에 할당된다.
**Stack**: 함수 인자 값, 함수 내의 지역 변수, 함수의 반환 주소 등이 저장되는 영역으로 함수 호출의 전반적인 처리와 리턴값을 가지고 있다. 상위 메모리 주소에서 하위 메모리 주소로 데이터가 저장된다.

# Java의 Stack과 Heap
Java 메모리 영역중 Stack 과 heap 영역의 사용에 초점을 맞춰 정리해보았다.

![](https://images.velog.io/images/dlrlejr132/post/6175d987-3778-4300-b74f-dd94389f33e5/image.png)

[Image from '[https://dzone.com/articles/java-memory-management](https://dzone.com/articles/java-memory-management)']


## Stack
**Heap 영역에 생성된 Object 타입의 데이터 참조값이 할당된다.
원시타입(primitive types)의 데이터가 값과 함께 할당된다.
지역변수들은 scope에 따른 visibility를 가진다.
각 Thread는 자신만의 Stack을 가진다.**

추가로 설명하자면 scope에 따른 visibility를 가진다는 뜻은 변수 scope에 대한 개념이다.
전역변수가 아닌 지역 변수가 foo()라는 함수에 할당 된 경우, 해당 지역변수는 다른 함수에서 접근 할 수 없다. 예를 들어, foo()라는 함수에서 bar() 함수를 호출하고 bar() 함수가 종료된 경우, bar() 함수 내부에서 선언한 모든 지역변수들은 stack에서 pop되어 사라진다.

위 이미지에서 스택을 보면 여러개가 존재하는 것처럼 Java에서 Stack 메모리는 Thread 하나당 하나씩 할당된다. 그러므로 Thread가 생성되고, 실행되는 순간 해당 Thread를 위한 Stack도 함께 생성되며, 각 Thread에서 다른 Thread의 Stack 영역에는 접근할 수 없다.

## Heap
**heap 영역에는 주로 긴 생명주기를 가지는 데이터들이 저장된다.(대부분의 오브젝트는 크기가 크고, 서로 다른 코드블럭에서 공유되는 경우가 많다.)
애플리케이션의 모든 메모리 중 stack에 있는 데이터를 제외한 부분이라고 보면 된다.
모든 Object 타입(Integer, String, ...)은 heap 영역에 생성된다.
몇개의 스레드가 존재하든 상관없이 단 하나의 heap 영역만 존재한다.
Heap 영역에 있는 오브젝트들을 가리키는 레퍼런스 변수가 stack에 올라가게 된다.**

```java
StringBulider builder = new StringBuilder();
```
위 코드에서 new 키워드의 역할은 생성하려는 오브젝트를 저장할 수 있는 충분한 공간이 Heap에 있는지 먼저 찾은 다음, StringBuilder type의 object를 heap 메모리에 생성하여 이것을 참조하는 builder 라는 변수를 Stack에 할당한다.
```java
String s = "hello";
s+= "world";
```
Java에서 String type은 immutable(불변객체) 이다. 어떤 연산을 수행할때마다 기존 오브젝트를 변경하는 것이 아니라 새로운 오브젝트를 생성하는 것이다. 위 코드에서 s+= "world" 은 heap에 "hello world"라는 String object가 새롭게 할당되는 작업이다. 

Java에서 Wrapper class에 해당하는 Integer, Chracter, Byte, Boolean, Double, Float, Short 클래스는 모두 Immutable이다. 그래서 heap에 있는 같은 object를 레퍼런스 하고 있는 경우라도, 새로운 연산이 적용되는 순간 새로운 오브젝트가 heap에 새롭게 할당된다.


## Garbage Collection
```java
        String url = "https://";
        url += "yaboong.github.io";
        System.out.println(url);
```
JVM의 Garbage Collector는 Unreachable Object를 우선적으로 메모리에서 제거하여 메모리 공간을 확보한다. Unreachable Object란 Stack에서 도달할 수 없는 Heap 영역의 객체를 말하는데, 위 코드에서 url에 처음 할당한 문자열 "https://" (String object)와 같은 경우를 말한다.

Garbage Collection 과정은 Mark and Sweep 이라고도 한다. JVM의 Garbage Collector 가 스택의 모든 변수를 스캔하면서 각각 어떤 오브젝트를 레퍼런스 하고 있는지 찾는과정이 Mark 다. Reachable 오브젝트가 레퍼런스하고 있는 오브젝트 또한 marking 한다. 첫번째 단계인 marking 작업을 위해 모든 스레드는 중단되는데 이를 stop the world 라고 부르기도 한다. (System.gc() 를 생각없이 호출하면 안되는 이유이기도 하다)

그리고 나서 mark 되어있지 않은 모든 오브젝트들을 힙에서 제거하는 과정이 Sweep 이다.

Garbage Collection 이라고 하면 garbage 들을 수집할 것 같지만 실제로는 garbage 를 수집하여 제거하는 것이 아니라, garbage 가 아닌 것을 따로 mark 하고 그 외의 것은 모두 지우는 것이다. 만약 힙에 garbage 만 가득하다면 제거 과정은 즉각적으로 이루어진다.

### 참고한 자료
[https://yaboong.github.io/java/2018/05/26/java-memory-management/](https://yaboong.github.io/java/2018/05/26/java-memory-management/)
[https://dzone.com/articles/java-memory-management](https://dzone.com/articles/java-memory-management)
[https://everybe-ok.tistory.com/15](https://everybe-ok.tistory.com/15)
