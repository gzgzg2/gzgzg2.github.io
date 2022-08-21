---
layout: post
title: "[Java Design Pattern] Spring을 활용한 Chain Of Responsibility"
categories:
- Java Design Pattern

tags:
- Java
- Gof
- 책임연쇄패턴
- 디자인패턴


toc: true
toc_sticky: true

date: 2022-08-21 
last_modified_at: 2022-08-21
comments: true
---

# Chain Of Responsibility Pattern (책임 연쇄 패턴) 
책임 연쇄 패턴은 `Gof Pattern 중 행동패턴`에 포함되며 요청을 하나이상의 체인으로 구성된 객체에게 전달하여 메세지를 보내는 대상과 처리하는 대상을 분리한다. 
체인은 책임에 따라 여러 역할로 나누고 각 체인은 서로 연결되어있다. 요청은 동적으로 처리되며, 하나의 체인은 책임에 따라 요청을 처리하고 역할이 끝나면 다음 체인으로 요청을 전달한다.   

## Diagrams
<img width="700" alt="image" src="https://user-images.githubusercontent.com/56028408/185785717-17cd4867-d508-4682-881e-d26c19ba8236.png">

### Client
Client는 요구사항에 따라 동적으로 체인을 구성한다. 체인은 하나일 수도 있고 여러개로 구성될 수 있다.  
이뿐만 아니라 발신자의 요청을 수신하여 체인에게 전달하는 역할도 담당한다.


### Handler 
모든 Concrete Handler에게 공통된 기능을 선언한 인터페이스  
요청을 처리하기 위한 메서드와 다음 체인을 지정하는 메서드가 포함되어 있다. 

### Concrete Handlers
Handler에 선언된 메서드를 구현한 클래스
요청을 수신하면 처리 여부에 따라 요청을 전달할지 판단한다.  
일반적으로 불변하며, 생성자를 통해 필요한 모든 데이터를 지정한다.  


```
적용하면 좋은 경우
1. 요청을 처리할 객체들이 "동적"으로 연결되어야 할 때
2. 요청을 처리할 수신자가 명시적으로 지정되어있지 않을 때
3. 요청의 발신자와 수신자를 분리하고 싶을 때
4. 코드에서 명시적으로 핸들러지를 지정하기 싫을 때 

장점
1. 분기문을 제거할 수 있음
2. 객체에게 유연하게 책임을 할당할 수 있음
3. 결합도 감소
    - 요청한 객체는 요청을 처리하는 객체를 직접적으로 참조하지 않음
    - 메시지를 보내는 대상과 받는 대상이 분리되어 결합도가 감소함  
4. 체인 내에서 구성을 변경하거나 삭제하여 책임을 동적으로 관리할 수 있음 (유연성 향상)

단점
1. 요청에 대한 신뢰성을 보장하지 않음 (체인에게 요청이 전달되지 않을 수 있음)
2. 시스템 성능에 영향이 갈 수 있음 (등록된 모든 체인을 순회함)
3. 코드를 확인하는 것이 쉽지않아 코드를 잘못 작성할 경우 체인 내에서 사이클이 발생할 수 있음
4. 디버깅으로 동작을 확인하기 어려움
```

# 적용 예제

## 가정
지하철 요금은 개찰구를 통과했을 때 지불하는 기본요금과 거리마다 추가되는 추가 요금으로 구성된다.  
개찰구를 통과할 지불하는 1250원과 10Km를 넘어가면 5Km 마다 100원씩 추가로 지불해야한다.  
10Km~50Km 구간에선 5Km 마다 100원씩 추가요금이 부과되고 50Km를 넘으면 8Km 마다 추가 요금이 부과된다.


## 조건문을 적용해서 해결한 예시
소프트웨어 적으로 지하철 요금을 계산하려면 기본 요금과 거리마다 부과되는 추가요금을 고려해야한다.  
가장 쉽게 구현할 수 있는 방법은 거리마다 분기문으로 처리하여 코드를 작성하는 것이다.
```java

 void calculateBaseOnDistance() {
        if (distance < 11) {
            System.out.println("기본요금");
        } else if (distance < 51) {
            System.out.println("5Km 마다 추가요금이 발생합니다.");
        } else if (distance > 51 ) {
            System.out.println("8Km 마다 추가요금이 발생합니다.");
        }
    }

```
조건문을 사용하여도 충분히 해결할 수 있지만 만약 새로운 요금 정책이 추가된다면  
정책마다 조건을 추가하여야 한다. 조건이 많아지면 코드의 가독성이 저하되고 유지보수가 힘들어진다.  
또한 많은 조건은 해당 클래스가 OCP를 위반한다는 신호로 받아드릴 수 있다.


## 책임 연쇄 패턴 적용
이제 조건문을 책임 연쇄 패턴을 이용하여 제거하는 방법을 알아보자 예제코드는 편리한 의존주입과 테스트 환경을 위해  
`Spring boot 2.6.3, junit5` 를 사용하였다.

### Diagrams
<img width="700" alt="image" src="https://user-images.githubusercontent.com/56028408/185790279-9e802f86-134c-45a0-a3ed-159427272e53.png">

### Client - version.1 (요청 전달 책임과 Chain 연결 책임을 분리)

**요청을 체인에게 전달**
```java
@Component
@RequiredArgsConstructor
public class FareChainCalculator {

    @Qualifier("basicFareCalculator")
    private final FareChain basicChain;

    public int operate(int distance) {
        return basicChain.calculateBaseOnDistance(distance);
    }

}
```

**체인의 연결 정보를 관리**
```java
@Configuration
public class FareChainConfig {

    @Bean
    public FareChain basicFareCalculator() {
        return new BasicFareCalculator(mediumDistancePassengerFareCalculator());
    }

    @Bean
    public FareChain mediumDistancePassengerFareCalculator() {
        return new MediumDistancePassengerFareCalculator(longDistancePassengerFareCalculator());
    }

    @Bean
    public FareChain longDistancePassengerFareCalculator() {
        return new LongDistancePassengerFareCalculator(null);
    }
}
```
`version.1`은 생성자 주입을 사용하여 체인을 연결한다.

### Client - version.2 (분리되지 않은 경우)

```java
public class FareChainCalculator {

    private final Map<String, FareChain> fareChainMap;

    public int operate(int distance) {
        setChain();
        FareChain basicChain = getBasicChain();
        return basicChain.calculateBaseOnDistance(distance);
    }

    private void setChain() {
        FareChain basicFareChain = getBasicChain();
        FareChain mediumDistancePassengerChain = getMediumDistanceFareChain();
        FareChain longDistancePassengerChain = getLongDistanceFareChain();

        basicFareChain.setNext(mediumDistancePassengerChain);
        mediumDistancePassengerChain.setNext(longDistancePassengerChain);
    }

    private FareChain getLongDistanceFareChain() {
        return fareChainMap.get(FareChainType.LONG_DISTANCE_PASSENGER_CHAIN.implementation());
    }

    private FareChain getMediumDistanceFareChain() {
        return fareChainMap.get(FareChainType.MEDIUM_DISTANCE_PASSENGER_CHAIN.implementation());
    }

    private FareChain getBasicChain() {
        return fareChainMap.get(FareChainType.BASIC_CHAIN.implementation());
    }
}
```
`version.2`은 Client가 요청 전달의 역할과 체인 연결 역할을 모두 수행한다. 해당 방법을 사용하려면  
Concrete Handlers가 FareChain의 `setNext(FareChain chain)` 메서드를 구현해야 한다. 

### Handler
예제의 클라이언트가 분리되지 않은 경우에서는 setNext 메서드도 정의해야 한다.
```java
public interface FareChain {

    int calculateBaseOnDistance(int distance);

    boolean canSupport(int distance);
}
```

### Concrete Handlers

**Concrete Handler.1**
```java
@AllArgsConstructor
public class BasicFareCalculator implements FareChain {
    private static final int MAX_DISTANCE = 10;
    private static final int BASE_FARE = 1250;

    private FareChain nextChain;

    @Override
    public int calculateBaseOnDistance(int distance) {
        if(!canSupport(distance)) {
            return BASE_FARE + /* 다음 체인 호출 */;
        }

        return BASE_FARE;
    }

    @Override
    public boolean canSupport(int distance) {
        return /* 요청 처리가 가능한지 판단 */;
    }
}
```

**Concrete Handler.2**

```java
@AllArgsConstructor
public class MediumDistancePassengerFareCalculator implements FareChain {
private static final int MAX_DISTANCE = 50;

    private FareChain nextChain;

    @Override
    public int calculateBaseOnDistance(int distance) {
        if (!canSupport(distance)) {
           return operate(distance) + /* 다음 체인 호출 */;
        }
        return operate(distance);
    }

    @Override
    public boolean canSupport(int distance) {
        return /* 요청 처리가 가능한지 판단 */;
    }

    private int operate(int distance) {
        return /* 계산식 */;
    }
}
```

**Concrete Handler.3**

```java
@AllArgsConstructor
public class LongDistancePassengerFareCalculator implements FareChain {

    private final FareChain fareChain;

    @Override
    public int calculateBaseOnDistance(int distance) {
        return /* 계산식 */;
    }

    @Override
    public boolean canSupport(int distance) {
        return /* 요청 처리가 가능한지 판단 */;
    }
}
```

### 결과 확인
이번 예제에서 구현한 책임 연쇄 패턴은 모든 체인을 순회하지 않고, 요청을 전달받은 체인이 요청을 처리할 수 없을 때
해당 체인의 역할을 수행하고 다음 체인에게 요청을 전달하는 방식으로 구현하였다.

```java
    /*
     * 거리가 10일 경우 기본요금 1250
     * 거리가 11일 경우 1350 (5km 마다 100원씩 추가요금 부여)
     * 거리가 51일 경우 2150 (8km 마다 100원씩 추가요금 부여)
     */
    @ParameterizedTest
    @CsvSource(value = {"10:1250", "11:1350", "51:2150"}, delimiter = ':')
    void calculate(String distance, String fare) {
        int extractFare = fareChainCalculator.operate(Integer.parseInt(distance));

        assertThat(extractFare).isEqualTo(Integer.parseInt(fare));
    }
```
<img width="211" alt="image" src="https://user-images.githubusercontent.com/56028408/185790183-4a18e24b-a4bf-4929-a59d-c824562b20b4.png">

해당 테스트를 실행하면 정상적으로 동작하는 것을 확인할 수 있다.



## 결론
예제의 시나리오는 Chain Of Responsibility Pattern 뿐만 아니라 다른 방법을 이용하여도 조건문을 제거할 수 있다.  
하지만 이번 게시글은 Chain Of Responsibility Pattern을 위한 글이므로 다른 방법은 소개하지 않았다.


---

# Reference
### **Hotspot** 
- https://refactoring.guru/design-patterns/chain-of-responsibility
= https://www.geeksforgeeks.org/chain-responsibility-design-pattern/









