---
layout: post
title: "[Java] synchronized"
categories:
- Java

tags:
- Java
- monitor
- synchronized
- Thread

toc: true
toc_sticky: true

date: 2022-06-26
last_modified_at: 2022-06-26
comments: true
---
# Java synchronized 
Java의 synchronized는 Thread간의 동기화 매커니즘이다. synchronized는 하나의 객체를 여러 스레드에서 동시에 사용하거나
static으로 선언한 객체를 여러 스레드에서 사용할 경우에 적용한다. 주의할 점은 synchronized는 객체 단위로 동작하기 때문에
한 객체에서 synchronized를 사용하는 여러개의 메서드가 있다면 해당 키워드가 적용된 메서드는 동시에 실행되지 않는다.
또한 내부에서 동기화를 위해 [**Monitor**](https://en.wikipedia.org/wiki/Monitor_(synchronization))를 사용하고 있으므로 성능저하 문제가 발생할 수 있다. 
 

## Monitor
모니터란 동시에 수행중인 프로세스나 스레드 사이에서 동기화를 적용하기 위한 **high-level synchronization construct** 이다.
모니터는 하나의 **lock**과 여러개의 **condition variable**로 이루어져있다. 모니터 내에서는 한번에 하나의 프로세스나 스레드만이 활동이 가능하므로
프로세스나 스레드가 모니터를 사용하다가 봉쇄상태로 변경되어도 다른 프로세스가 모니터에 접근하지 못한다.

## Java Monitor
> **📌 모니터는 synchronized를 사용했을 때 이용된다.** 

Java의 모든 객체는 Monitor를 보유하고 있다. 객체는 heap 영역에 존재하므로 모든 스레드에서 공유가 가능하기 때문에 Monitor는 객체에 한번에 하나의 스레드만 접근할 수 있도록 막는다. 
그리고 객체는 모니터가 할당되지 않은 상태이면 호출된 스레드에게 모니터를 할당한다. 소유권을 갖게된 스레드는 모니터에 진입하게 된다. 모니터에 하나의 스레드가 진입하게 되면 다른 스레드는
모니터를 사용할 수 있을 때까지 대기하게 된다. 즉 오직 모니터를 소유한 스레드만이 임계구역에서 작업을 할 수 있는 것이다.

## 1. 공유객체에 접근할 때 synchronized를 사용하지 않았을 경우

### 기부금을 처리하는 단체
```java
public class Contribution {
    private int amount;
    public void donate() {
        this.amount++;
    }
    public int getTotal() {
        return amount;
    }
}
```
단순하게 기부금 필드와 기부금을 축적하는 메서드만 보유하고 있는 클래스이다. 

### 기부자
```java
public class Contributor extends Thread {
    private Contribution contribution;
    private String name;

    public Contributor(Contribution contribution, String name) {
        this.contribution = contribution;
        this.name = name;
    }

    public void run() {
        for(int loop=0; loop<1000; loop++) {
            contribution.donate();
        }
        System.out.format("%s total=%d\n", this.name, contribution.getTotal());
    }
}
```
1인당 1원씩 1000번 기부하고 기부가 완료되면 쌓인 기부금을 출력하는 클래스이다.

### 실행 소스코드
```java
public class App {

    public static void main(String[] args) {
        Contributor[] contributors = new Contributor[10];
        Contribution contribution = new Contribution();

        for(int loop=0; loop < 10; loop++) {
            contributors[loop] = new Contributor(contribution, "Contributor"+loop);
        }

        for(int loop=0; loop < 10; loop++) {
            contributors[loop].start();
        }
    }
}
```
하나의 단체에 10명의 기부자가 1000원씩 기부하는 App 클래스이다.  
소스코드로 예상할 수 있는 결과의 최대값은 10,000 일 것이다. 결과를 확인해보자.

### 결과
![스크린샷 2022-06-26 오후 11 27 55](https://user-images.githubusercontent.com/56028408/175819107-4f2f559c-e58e-4491-877b-eb90d68d0ff2.png)

결과는 우리가 예상한 값과 다르다. 물론 여러번 실행하면 10,000 이라는 값이 출력되기도 한다. 그렇지만 대부분 10,000이라는 값은 출력되지 않을 것이다.  
이러한 문제가 발생하는 이유는 여러개의 contributors 객체에서 하나의 contribution 객체의 donate() 메서드에 접근하기 때문이다.   
앞서 말했듯이 객체는 heap 영역에 생성되므로 모든 스레드가 접근할 수 있다. 모든 스레드가 동시에 공유영역에 접근할 수 있다는 것은 스레드 동기화 문제가 발생할 수 있다는 것과 같다.

## 2. 문제 해결 방법
### synchronized
위와 같은 경우 문제 해결방법은 간단하다. **donate()** 메서드에 synchronized 를 사용하면 된다.

```java
     public synchronized void donate() {
        this.amount++;
    }
```

### 결과
![스크린샷 2022-06-26 오후 11 39 14](https://user-images.githubusercontent.com/56028408/175819627-034845b1-3155-40a4-b08a-c1c79bd09982.png)

synchronized 메서드를 추가하면 donamte() 메서드에 하나의 스레드만이 접근할 수 있게되므로 여러번 실행하여도 최대값은 10,000이 출력될 것이다.
하지만 이 예시는 간단한 예시이기 때문에 동기화가 필요한 메서드에 무조건 synchronized를 사용하는 방법은 옳지 않다.
메서드 내부에서 동기화가 필요하지 않은 라인도 동기화가 적용되어 성능에 큰 영향을 끼칠 수 있기 때문이다. 이번 게시물에서 소개하진 않았지만 인스턴스 메서드안에 동기화 블록을 적용하는 방법도 존재한다.

## 결론
synchronized를 사용해야할 경우는 존재할 수 있다. 그렇지만 synchronized의 동작 방식을 모르는 상태로 사용하게 되면  
큰 성능이슈를 겪을 수 있다. 뿐만 아니라 클래스 변수는 인스턴스 메서드의 synchronized가 적용되지 않기 때문에 주의해서 사용해야 한다. 


# Reference
### Monitor 
- [https://gzgzg2.github.io/os/2022/04/02/os-study-07.html#4-monitor](https://gzgzg2.github.io/os/2022/04/02/os-study-07.html#4-monitor)
- [https://en.wikipedia.org/wiki/Monitor_(synchronization)](https://en.wikipedia.org/wiki/Monitor_(synchronization))
- [https://ssup2.github.io/programming/Java_Monitor_synchronized/](https://ssup2.github.io/programming/Java_Monitor_synchronized/)

### Java Monitor
- [https://ssup2.github.io/programming/Java_Monitor_synchronized/](https://ssup2.github.io/programming/Java_Monitor_synchronized/)
- [https://www.artima.com/insidejvm/ed2/threadsynch.html](https://www.artima.com/insidejvm/ed2/threadsynch.html)

### Java synchronized
- 자바 성능 튜닝 이야기 :: story.8 synchronized는 제대로 알고 써야한다. 
















