---
layout: post
title: "[Java] Getter Setter 필요없을까?"
categories:
- Java

tags:
- Java
- Getter, Setter
- OOP

toc: true
toc_sticky: true

date: 2022-06-12
last_modified_at: 2022-06-28
comments: true
---

## 들어가며
이전에 무지성으로 남발하여 주로 사용하였던 Getter Setter가 객체지향 세계에선 아주 지양해야할 패턴이라는 것을 알게 되었다.
Getter Setter가 존재하는 이상 그 클래스는 능도적인 객체가 아닌 자료구조에 불과하다고 한다. 그런데 가끔 특정한 라이브러리에서 Getter Setter 를 필요로 하는 것을 느꼈다.
물론 옵션을 설정할 수 있지만 주의해야 하는 것은 마찬가지이다. 그래서 이번에는 `Java Beans Pattern`에서 벗어나고 싶은데, 완전히 자유로워지기 힘든 이유를 정리하고자 한다.

----

## Java Beans Pattern 지양해야 하는 이유
자바 개발자라면 객제지향 프로그래밍의 4대 원칙을 한번쯤 들어봤을 것이다. 간단하게 소개하자면 아래와 같다.

### **OOP 4대 원칙**
**1. 추상화(Abstraction)**
- 구체적인 것을 `분해` 하여 `재조립` 하는 것

**2. 상속(Inheritance)**
- `상위 클래스`의 특성을 `하위 클래스`가 물려받는 것

**3. 캡슐화, 정보은닉 (Encapsulation)**
- 역할에 관련있는 객체의 속성과 실제 행위를 묶어 실제 구현 내용을 외부에 노출하지 않는 것

**4. 다형성(Polymorphism)**
- 같은 형태를 가지고 있지만 다른 행위를 할 수 있음

### 1. 캡슐화 위배
Getter와 Setter의 경우 캡슐화를 위반한다. 객체가 무엇을 캡슐화 하고 있는지와 객체 내부의 복잡성은 오직 객체만이 알고 있어야 한다.
그런데 Getter와 Setter는 외부에 객체의 데이터를 노출하게 만든다. 물론 필드에 직접 접근하는 것이 아닌 메서드를 이용하여 접근하는 것이지만 
우리는 Getter가 데이터에 직접 접근하여 해당 필드의 데이터만을 반환할 것이라고 믿고있다.


>객체지향 세계에선 객체에게 특정한 행위를 요청했을 때, 외부에서는 객체가 어떠한 재료들로 무엇을 만드는지 알아선 안되고 알 필요도 없다.
{: .prompt-info}

### 2. 객체를 자료구조로 전락시킨다
객체지향 세계에서의 객체는 살아있고 능동적이다. 그런데 Getter/Setter가 남발된 객체는 생명이 없다. 그저 데이터를 외부에 노출하는 단순 자료구조일 뿐이다.


```java
//안 좋은 예시
class Cash {
  private int value; 

  public  int getDollars() {
    return this.value();
  }
}

// 좋은 예시
class Cash {
  private final int value;

  public int dollars() {
    return this.value;
  }
}
```
차이는 메서드 이름밖에 없다고 생각할 수 있다. 그렇지만 메서드 이름에서 가져오는 뉘앙스는 완전히 다르다.  
앞에서 설명했듯 객체는 살아있고 능동적이다. 그런데 getDollars()는 객체에게 달러를 찾아 반환하라고 `명령` 한다.  
반면 dollars()의 경우 객체에게 얼마나 많은 달러가 필요한지 묻는다. 객체를 생명체로서 존중하고 있는 것이다.  
또한 dollars()는 객체가 요청을 듣고 반환할 때 외부에선 객체가 어떠한 재료와 행위로 값을 반환하는지 유추하기 힘들다.


### 3. 불변객체가 될 수 없다
<kbd>Setter</kbd> 를 사용하면 누구나 객체의 값을 쉽게 수정할 수 있다. 이는 곧 불변객체가 될 수 없다는 의미와 같다.
불변객체는 생성과 동시에 값을 초기화 하므로 개발자가 이후에 객체의 상태값을 변경할 수 없다. 값이 변하지 않으면 `식별자 가변성`  문제와 `부수효과` `시간적 결합` 문제에서 자유롭다. 뿐만 아니라 불변객체는 `실패 원자성` 의 장점이 있다. 그런데 <kbd>Setter</kbd>를 사용하게 되면 이러한 장점을 전부 누리기 어려워진다.


>객체의 값을 쉽게 수정할 수 있게 된다면 개발자의 실수로 의도하지 않았던 결과를 초래할 수 있다. 문제를 해결하기 위해 실수로 발생한 부수효과(side effect)를 제거하려면 값을 수정하는 코드를 전부 찾아서 디버깅 해야하기 때문에 많은 시간이 소요된다.
{: .prompt-info}


----

## Getter, Setter가 없을 때 발생하는 이슈
앞에서 설명한 내용들을 보면 java beans pattern 은 정말 안티패턴 처럼 느껴진다. 나만 그럴수도,,?
여튼 그래서 실제로 업무를 진행하면서 Setter 뿐만 아니라 Getter도 생성하지 않고 개발했던 적이 있었다.  자주 사용하던 방법이 아니여서 익숙하진 않았지만 크게 불편함은 없었다. 그런데 .. 예상하지도 못했던 부분에서 문제가 발생하였다. ObjectMapper로 Json을 역직렬화 할 때 멤버변수를 찾지 못하여 실패하는 이슈가 발생한 것이었다.

### 1. ObjectMapper
ObjectMapper에서 JSON을 자바 객체로 역직렬화 할 때 getter, setter 메서드 이름의 "get"과 "set"을 제거하여 객체의 멤버변수를 찾는다. getter, setter 없이 다른 방법으로도 필드명을 알아낼 수 있지만 따로 옵션을 설정하거나 어노테이션을 지정해줘야 한다.

**📌 @JsonProperty**
```json
{
  "dollars" : 100
}
```
```java
class Cash {

  @JsonProperty("dollars")
  private Integer dollars;

}
```

**📌 ObjectMapper 설정 변경**
```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
```



<br>

### Reference

---
- [https://jenkov.com/tutorials/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields](https://jenkov.com/tutorials/java-json/jackson-objectmapper.html#how-jackson-objectmapper-matches-json-fields-to-java-fields)  
- [https://bactoria.github.io/2019/08/16/ObjectMapper%EB%8A%94-Property%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B0%BE%EC%9D%84%EA%B9%8C/](https://bactoria.github.io/2019/08/16/ObjectMapper%EB%8A%94-Property%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B0%BE%EC%9D%84%EA%B9%8C/)















