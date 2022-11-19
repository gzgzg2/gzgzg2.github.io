---
layout: post
title: "[Java Design Pattern] Strategy Pattern"
categories:
- Java Design Pattern

tags:
- Java
- Gof
- 전략패턴
- Strategy Pattern
- 디자인패턴


toc: true
toc_sticky: true

date: 2022-11-19 
last_modified_at: 2022-11-19
comments: true
---

# Strategy Pattern (책임 연쇄 패턴) 
전략패턴 패턴은 `Gof Pattern 중 행동패턴`에 포함되며 런타임 시 알고리즘을 변경할 수 있다.
일반적으로 알고리즘을 정의하는 인터페이스를 하나 두고 알고리즘을 구현하는 여러개의 구현체로 구성된다.

## Diagrams
<img width="400" alt="image" src="https://user-images.githubusercontent.com/56028408/202852384-9a158fe6-8cf2-4aea-820b-8905e6be5ee0.png">

### Context
전략을 포함하고 있는 클래스 컨텍스트는 동작을 구현하는 책임을 `Strategy`에 위임한다.
실제로 컨텍스트가 전략 변경이 필요한 클래스라고 볼 수 있다.
일반적으로 `Strategy`를 변경할 수 있는 메소드도 포함하고 있다.  

### Strategy
말 그대로 전략이다. `Context`가 필요로 하는 행위를 정의한 `interface`이다.
`Strategy`은 컨텍스트에 영향을 주지 않고 행동을 변경할 수 있도록 `interface`로 정의하여 구현한다.
이를 `Context`의 필요에따라 여러가지 전략으로 구현한다. 

### Concrete Strategy
`Strategy`에 선언된 메서드를 구현한 클래스
  


```
적용하면 좋은 경우
1. 런타임시에만 어떠한 알고리즘이 필요한지 알 수 있는 경우
2. 클라이언트와 알고리즘을 독립적으로 구분하고 싶을 경우
3. 특정 타입이나 값에 따라서 알고리즘이 변경되는 경우

장점
1. 런타임까지 사용할 전략에 대한 결정을 연기하면 호출 코드가 보다 유연하고 재사용이 가능해진다.
2. 알고리즘을 사용하는 Context와 독립적으로 알고리즘을 변경할 수 있다.
3. 클라이언트와 알고리즘의 결합도 감소 (인터페이스의 정의가 변경되지 않으면 클라이언트는 영향을 받지않음) 

단점
1. 아직까진 떠오르지 않는 거 같음 .... 
```

# 적용 예제

## 가정
피카츄가 포켓몬 배틀에 참여하려고 한다. 피카츄는 하나의 스킬만 가질 수 있으며 스킬은 라운드가 바뀔 때마다
변경할 수 있다. 피카츄가 배틀이 시작됐을 때 자유롭게 스킬을 변경할 수 있도록 도와주자.


### Diagrams
<img width="400" alt="image" src="https://user-images.githubusercontent.com/56028408/202852343-09555cb5-459c-4f6b-ab9a-71de4654a023.png">

### Context - (전략을 필요로 하는 대상)

```java
public class Pikachu {
    private Attack attack;

    public Pikachu(Attack attack) {
        this.attack = attack;
    }

    public String conductingBattle() {
        return attack.doAttack();
    }

    public void changeAttack(Attack newAttack) {
        this.attack = newAttack;
    }
}
```
간단하게 전략을 사용하는 메소드만 포함하고 있다. 

### Strategy
```java
public interface Attack {
    String doAttack();
}
```
`doAttack()`메소드만 포함하고 있는 간단한 전략이다.
`java8` 이상이면 인터페이스 메소드가 하나일 경우 간단한 문법으로 인터페이스 구현체를 생성할 수 있다.
`토비의 스프링3.0`에서는 전략의 메소드가 하나일 경우 `Template/callback 패턴`이라고도 부르는 것 같다. 
주의할 점은 `Template Method Pattern`과는 아예 다른 패턴이니 혼동하면 안된다는 것이다. 

### Concrete Strategy

**Concrete Strategy.1**
```java
public class HeadbuttAttack implements Attack {
    @Override
    public String doAttack() {
        return "몸통 박치기 !!";
    }
}

```

**Concrete Strategy.2**

```java
public class IronTailAttack implements Attack {
    @Override
    public String doAttack() {
        return "아이언 태일!!";
    }
}

```

**Concrete Strategy.3**

```java
public class MillionVoltsAttack implements Attack {
    @Override
    public String doAttack() {
        return "백만볼트 !!";
    }
}

```

### 배틀 시작
```java
  public class PokemonBattle {
    public static void main(String[] args) {
        Map<String, Attack> attacks = createAttacks();

        System.out.println("Round 1 Start !!");
        Pikachu pikachu = new Pikachu(attacks.get("headButt"));
        System.out.println(pikachu.conductingBattle());

        System.out.println("Round 2 Start !!");
        pikachu.changeAttack(attacks.get("ironTail"));
        System.out.println(pikachu.conductingBattle());

        System.out.println("Round 3 Start !!");
        pikachu.changeAttack(attacks.get("millionVolt"));
        System.out.println(pikachu.conductingBattle());
    }

    private static Map<String, Attack> createAttacks() {
        return  Map.of(
                "headButt", new HeadbuttAttack(),
                "ironTail", new IronTailAttack(),
                "millionVolt", new MillionVoltsAttack()
        );
    }
}


```

### 결과 확인
<img width="174" alt="image" src="https://user-images.githubusercontent.com/56028408/202853007-677a0e52-aaf2-4f9b-93a1-d3b24fa93809.png">

피카츄가 원하는 대로 라운드가 변경될 때마다 스킬을 사용할 수 있게 됐다.
예제가 너무 간단해서 굳이 전략패턴을 사용하지 않아도 구현이 가능했지만,, 전략패턴의 의도가 무엇인지 확인하는 용도로 보면 좋을 것 같다.


### Lambda로 전략 생성하기
```java
 Pikachu pikachu = new Pikachu(() -> "몸통 박치기 !!");
        pikachu.changeAttack(() -> "아이언 태일 !!");
        pikachu.changeAttack(() -> "백만볼트 !!");
```
인터페이스의 메소드가 하나인 경우 위처럼 간단하게 구현체를 생성할 수 있다. 

---

# Reference
### **Chain Of Responsibility Pattern** 
- <https://refactoring.guru/design-patterns/strategy/java/example>
- <https://en.wikipedia.org/wiki/Strategy_pattern>
- <https://www.baeldung.com/java-strategy-pattern>









