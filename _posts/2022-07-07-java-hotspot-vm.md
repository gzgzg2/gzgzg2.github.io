---
layout: post
title: "[Java] Hotspot JVM & JIT"
categories:
- Java

tags:
- Java
- hotspot 
- jvm
- jit

toc: true
toc_sticky: true

date: 2022-07-07 
last_modified_at: 2022-07-07
comments: true
---
# Hotspot JVM
Hotspot JVM은 JVM 구현의 일종이으로 높은 확장성과 성능을 제공한다. 자바 1.3 버전부터 기본 VM으로 사용되어 왔기 때문에 현재 운영되는 대부분의 자바
애플리케이션은 Hotspot 기반의 VM 이라고 볼 수 있다. Hotspot VM에는 런타임시 동적으로 최적화를 수행하는 `JIT`가 포함되어 있다. JIT 뿐만 아니라 다음과 같은 다양한 기능을 제공한다.

```text
- GC를 이용하여 자바의 Heap 메모리를 효율적으로 관리
- 프로파일링, 모니터링 및 디버깅 도구
- 병렬 컴파일을 지원하는 컴파일러 인터페이스
- 네이티브 스레드 기반의 선점형 멀티스레딩
- 초고속 스레드 동기화
- VM 런타임
```

---

## Hotspot JVM 구조
![스크린샷 2022-07-07 오후 3 22 16](https://user-images.githubusercontent.com/56028408/177789303-3e075557-0df2-47a9-b527-e52769620796.png)
{: width="500" height="500" }

Hotspot JVM는 크게 `Class Loader`, `Runtime Data Area`, `Execution Engine`으로 구성되어 있다.  
여기서 세부 항목인 `Heap`, `JIT Compiler`, `Garbage Collection`은 성능과 관련된 JVM의 주요 구성요소이다.

### ClassLoader
클래스로더는 **JRE**의 핵심 구성요소로 **런타임** 동안 자바 클래스를 `로딩, 링크, 초기화` 절차를 거쳐 **JVM에 동적으로 로드**하는 역할을 한다.   
일반적으로 클래스들은 한번에 모두 로딩되지 않고 애플리케이션이 필요로 할 때 로딩한다. 또한 클래스로더는 다음과 같은 세 가지 원칙을 따른다.


**📌 Delegation**

- 클래스 로딩 요청을 상위 클래스 로더에게 전달하는 것. 상위 클래스에서 클래스를 찾거나 로드하지 않는 경우에만 하위 클래스 로더가 클래스를 로드한다.

**📌 Visibility**

- 하위 클래스 로더는 상위 클래스 로더가 로드한 모든 클래스를 볼 수 있다. 그러나 부모 클래스로더는 자식 클래스 로더가 로드한 클래스를 볼 수 없다.

**📌 Uniqueness**

- 클래스는 한번만 로드할 수 있다. 상위 클래스에서 이미 로드한 클래스를 하위 클래스로더가 다시 로드할 수 없다.


### Runtime Data Area


### Execution Engine

---

## JIT(Just In Time)

### Client

### Server

---




# Reference
### Hotspot 
- [https://www.tuning-java.com/195](https://www.tuning-java.com/195)
- [https://velog.io/@aki/HotSpot-JVM](https://velog.io/@aki/HotSpot-JVM)
- [https://www.oracle.com/java/technologies/whitepaper.html](https://www.oracle.com/java/technologies/whitepaper.html)
- [https://velog.io/@roo333/Java-Garbage-Collection-Basics] (https://velog.io/@roo333/Java-Garbage-Collection-Basics)
- 자바 성능 튜닝 이야기 :: story.16 JVM은 어떻게 구동될까? 

### ClassLoader
- [https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/](https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/)
- [https://docs.oracle.com/javase/7/docs/api/java/lang/ClassLoader.html](https://docs.oracle.com/javase/7/docs/api/java/lang/ClassLoader.html)
- [https://joomn11.tistory.com/14](https://joomn11.tistory.com/14)
- [https://www.javatpoint.com/classloader-in-java](https://www.javatpoint.com/classloader-in-java)












