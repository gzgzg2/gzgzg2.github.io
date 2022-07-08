---
layout: post
title: "[Java] Hotspot JVM 맛보기"
categories:
- Java

tags:
- Java
- hotspot 
- jvm
- jit
- 자바성능튜닝이야기

toc: true
toc_sticky: true

date: 2022-07-07 
last_modified_at: 2022-07-07
comments: true
---
> 이번 포스팅은 JVM에 대해 간단하게 정리한 포스팅 입니다.  
> 각 모듈에 대한 자세한 정보는 다음 포스팅에서 다루겠습니다 ! 

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


**1. Delegation**
- 클래스 로딩 요청을 상위 클래스 로더에게 전달하는 것. 상위 클래스에서 클래스를 찾거나 로드하지 않는 경우에만 하위 클래스 로더가 클래스를 로드한다.

**2. Visibility**
- 하위 클래스 로더는 상위 클래스 로더가 로드한 모든 클래스를 볼 수 있다. 그러나 부모 클래스로더는 자식 클래스 로더가 로드한 클래스를 볼 수 없다.

**3. Uniqueness**
- 클래스는 한번만 로드할 수 있다. 상위 클래스에서 이미 로드한 클래스를 하위 클래스로더가 다시 로드할 수 없다.


### Runtime Data Area
Runtime Data Area는 JVM이 실행 중에 사용되는 다양한 데이터 영역을 정의한다. 데아터 영역 중 일부는 JVM 시작시 생성되고 JVM이 종료될 때만 소멸된다.
Runtime Data Area는 스레드가 `공유하는 영역`과 `공유하지 않는 영역`으로 나눌 수 있다.

**1. 스레드 공유 데이터 영역**
- Heap and Direct Memory, Method Area 
- 해당 영역은 JVM 시작 시 생성되고 모든 스레드에서 공유한다.
- GC는 Heap 영역에서만 발생한다.
 
**2. 스레드가 공유하지 않는 데이터 영역**
- PC Register, JVM Stack, Native Method Stack
- 해당 영역은 스레드가 생성될 때 생성되고 해당 스레드에서만 사용된다. 
- 여러 스레드가 실행 중인 경우 이러한 데이터 영역의 여러 복사본이 생성된다.

### Execution Engine
실행엔진(Execution Engine)은 로딩된 클래스의 메서드들에 포함된 모든 인스트럭션 정보를 실행한다.
이때 바이트코드로 작성되어 있는 인스트럭션 정보를 OS가 읽을 수 있는 형태(기계어)로 변환해야 하는데, 실행엔진은 바이트코드를 한 줄마다 읽으며 변환하는 작업을 거친다. 
실행엔진은 Java 코드 실행 뿐만 아니라 최적화, 메모리관리 또한 담당한다.

Java가 OS에 독립적일 수 있는 이유는 이처럼 런타임 시에 바이트코드를 기계어로 변경하기 때문이다. 실행 엔진은 앞서 소개한 최적화, 메모리 관리, 실행 등을 담당하는 세 가지 구성 요소가 포함되어 있다.

**1. 인터프리터**

바이트 코드를 읽고 기계어로 변환하여 순차적으로 실행한다. 인터프린터가 매번 동일한 바이트코드를 여러번 변환하는 것은 성능을 저하시키는 원인 중 하나이기 때문에
이러한 문제를 해결하기 위해 `JIT 컴파일러`가 도입되었다.

**2. JIT(Just In Time)**

인터프린터의 단점을 보완하기 위해 도입된 컴파일러이다. JIT 컴파일러의 주요 목적은 성능을 향상시키는 것이다.
모든 코드는 첫 실행시 인터프리터에 의해서 시작되고, 해당 코드가 자주 호출되면 컴파일할 대상이 된다. 
해당 메서드가 정상적으로 해석되면 JIT 컴파일러가 `수행 카운터`를 증가시킨다. JIT 컴파일러는 `수행 카운터`와 `백에지 카운터` 값으로
컴파일할 대상을 정하기 떄문에 `수행 카운터`가 높거나 `백에지 카운터`가 높은 메서드를 우선적으로 네이티브 메서드로 변환한다.
JIT 컴파일러에 의해 변환된 메서드는 다시 호출될 때 변환 과정을 거치치않고 바로 실행되기 때문에 성능이 향상된다.

간단하게 설명하면 <mark>자주 호출되는 메서드(Hotspot)</mark>를 우선적으로 컴파일한다.

**3. GC(Garbage Collection)**

GC(Garbage Collection)는 `메모리(heap)`를 관리하는 자바 프로그램이다. 런타임에 `참조되지 않는 객체의 메모리를 회수`하기 때문에 자바 메모리를 효율적으로 관리할 수 있다.
메모리를 GC가 관리하기 때문에 개발자는 메모리를 할당하거나 해제하는 로직을 작성할 필요가 없다. 추가로 GC는 항상 백그라운드에서 실행된다.

<br>

---

# Reference
### **Hotspot** 
- [https://www.tuning-java.com/195](https://www.tuning-java.com/195)
- [https://velog.io/@aki/HotSpot-JVM](https://velog.io/@aki/HotSpot-JVM)
- [https://www.oracle.com/java/technologies/whitepaper.html](https://www.oracle.com/java/technologies/whitepaper.html)
- [https://velog.io/@roo333/Java-Garbage-Collection-Basics](https://velog.io/@roo333/Java-Garbage-Collection-Basics)
- 자바 성능 튜닝 이야기 :: story.16 JVM은 어떻게 구동될까? 

### **ClassLoader**
- [https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/](https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/)
- [https://docs.oracle.com/javase/7/docs/api/java/lang/ClassLoader.html](https://docs.oracle.com/javase/7/docs/api/java/lang/ClassLoader.html)
- [https://joomn11.tistory.com/14](https://joomn11.tistory.com/14)
- [https://www.javatpoint.com/classloader-in-java](https://www.javatpoint.com/classloader-in-java)

### **Runtime Data Area**
- [http://www.herongyang.com/JVM/Data-Area-What-Are-Runtime-Data-Areas.html](http://www.herongyang.com/JVM/Data-Area-What-Are-Runtime-Data-Areas.html)
- [https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html)

### **Execution Engine**
- [https://www.geeksforgeeks.org/execution-engine-in-java/](https://www.geeksforgeeks.org/execution-engine-in-java/)
- [https://joomn11.tistory.com/17](https://joomn11.tistory.com/17)









