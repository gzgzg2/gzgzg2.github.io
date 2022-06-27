---
layout: post
title: "[네트워크 스터디] Chapter_04 엑세스 회선을 통해 인터넷의 내부로 !"
categories:
  - networkStudy
tags:
  - 인터넷
  - 액세스 회선
  - ADSL 
  - LAN

toc: true
toc_sticky: true

date: 2022-06-08
last_modified_at: 2022-06-08
comments: true
---

## ADSL 기술을 이용한 엑세스 회선의 구조와 동작

### **✔️  인터넷의 기본은 가정이나 회사의 LAN과 같다**

- 인터넷은 거대하고 복잡한 시스템이지만 패킷을 중재하는 부분은 가정, 회사의 LAN과 동일하다.
- 라우터의 기본적인 구조나 동작도 전혀 다르지 않다
- 가정이나 회사의 네트워크 규모가 커진 것이 인터넷

### **✔️ 가정, 회사의 네트워크와 인터넷의 차이점**

- 인터넷은 중계 장치의 거리가 매우 멀다.
- 대한민국과 미국을 연결하려면 태평양을 넘어야하기 때문에 이더넷 케이블로 연결은 어렵다.
- 라우터의 등록정보가 수시로 바뀌기 때문에 경로 등록을 자동화 해야한다.
    - 통신 장애가 발생하거나 인터넷에 새로 접속하는 회사가 있을 때마다 변경하는 것은 무리가 있음
- 요약하자면 거리의 차이와 경로 정보의 등록 방법 차이가 인터넷과 회사, 가정 LAN과의 차이

### ✔️ 사용자 인터넷을 연결하는 액세스 회선

- 인터넷 접속용 라우터는 액세스 회선의 규칙에 따라 패킷 송신 동작을 실행한다. (이더넷의 규칙과 다른 부분)
- 일반 가정에선 ADSL, FTTH, CATV, ISDN 등을 액세스 회선으로 사용한다.

```text
💡 액세스 회선은 인터넷과 가정, 회사의 LAN을 연결하는 통신회선이다. 
💡 액세스 회선은 통신 회선의 사용법을 나타내므로 통신 회선의 원리나 구조를 나타내는 것이 아니다. 
```

### ✔️ADSL 모델에서의 패킷 신호화 과정

[**ADSL 기술을 사용한 액세스 회선의 구성도]**

![스크린샷 2022-05-04 오전 9 44 48](https://user-images.githubusercontent.com/56028408/166886670-ba59b0f3-34a8-4980-9dfb-a1c4571b0ea3.png)

위 그림의 오른쪽에서 왼쪽으로 패킷이 흘러간다. 즉 사용자의 라우터에서 송신된 패킷은 ADSL 모뎀이나 전화의 케이블을 통해 전화국에 도착하며 도착한 패킷은 ADSL 사업자의 네트워크를 경유하여 프로바이더 (ISP: Internet Service Provider)에 도착하는데, 이때 패킷은 도착하는 동안 여러 형태로 모습을 바꾼다.

```text
💡 ADLS(Asymmetric Digital Subscriber)모델이란 전주에 부설된 전화용 금속제 케이블을 이용하여
  고속으로 통신하는 기술의 일종으로 사용자로부터 인터넷으로 향하는 상향(업로드)과 인터넷에서 사용자로 향하는 
  하향(다운로드)의 통신 속도가 서로 다른 것을 가리킵니다.
```

**[액세서 회선을 통한 패킷의 전송 과정]**

![스크린샷 2022-05-04 오전 9 55 29](https://user-images.githubusercontent.com/56028408/166886653-c4bdfeaa-6bda-43e8-945f-9dc54eb1ced5.png)


```text
(1~4) 위 그림의 4번까지는 이더넷의 라우터와 큰 차이가 없음
(5) 인터넷 접속용 라우터와 **ADSL** 모뎀이 이더넷으로 연결되어 있으면 이더넷의 규칙에 따라 패킷을 송수신
(5) 이더넷의 헤더가 액세스 회선 사업자에따라 변형된다.
(5) MAC헤더 PPoE헤더 PPP헤더라는 3개의 헤더가 추가로 붙고 이더넷의 규칙따라 신호를 변환하여 송신한다
(6) 인터넷 접속용 라우터가 송신한 패킷을 ADSL 모뎀이 받아서 작게 분할하여 **셀**에 저장한다.
(7) 분할된 셀은 ATM 이라는 통신 기술에 사용된다.
(8) 셀에 저장된 패킷을 신호로 변환한다. 변환할 때는 직교 진폭 변조(QAM)라는 방식을 사용한다.
(9) 전화 케이블을 통해 전화국에 도착한 신호가 배선반과 스플리터를 통과하여 **DSLAM** 에 도착한다.
(10) DSLAM이 신호의 파형을 읽어 진폭과 위상을 조사하고 이것이 어느 비트에 대응하는지 판단하여 디지털 신호로 복원하여 전송한다.
(11) ATM 인터페이스가 셸을 수신한다.
(12) ATM 인터페이스가 셸을 원래의 패킷 형태로 되돌린다. (ATM 인터페이스를 셸을 원래 패킷으로 되돌리는 기능이 있음)
(13) BAS에 건내주기 위한 용도로 사용된 MAC 헤더와 PPoE헤더를 제거하고 PPP 헤더 이후 부분을 추출한다.
(14) 터널링에 필요한 헤더를 붙여서 출구를 향해 중계한다.
(15) 패킷이 터널링 출구에 있는 터널링용 라우터에 도착한다.
(16) 터널링용 헤더를 분리하고 IP 헤더 이후 부분의 패킷을 추출한다.
(17) 인터넷 내부에 중계한다.

💡 ADSL 모뎀은 패킷을 셀로 분할하고 전기 신호로 바꾸어 스플리터에 송신한다.
💡 셀은 맨 앞부분에 5바이트의 헤더와 48바이트의 데이터가 이어지는 작은 디지털 데이터의 덩어리이다.
💡 DSLAM이란 다수의 ADSL 모뎀의 기능을 여러개로 묶어서 하나의 케이스에 넣은 것
💡 DSLAM은 ATM 인터페이스를 가지고 있고 패킷을 분할한 셸 형태 그대로 후방의 라우터와 주고받는다. 
💡 BAS는 ATM 셸을 패킷으로 복원하여 인터넷 내부에 중계한다.
```

## 액세스 회선으로 이용하는 PPP와 터널링

> 내용을 설명하기전 다시한번 짚어보자면, ADSL이나 FTTH 등의 액세스 회선에서 인터넷을 향해 흘러온 패킷은 액세스 회선을 소유한 사업자가 운영하는 BAS에 도착한다.
>
>
> 인터넷은 다수의 라우터를 함께 연결하여 만든 것으로 액세스 회선을 라우터에 연결하는 것이 원칙이다. 그리고 액세스 회선이 ADSL이나 FTTH로 진화한 것으로 이것에 맞게 액세스 회선을 연결하는 라우터도 진화했다. 진화한 라우터를 BAS 라고 한다. 아래 내용은 무엇이 진화했는 지 설명하는 내용이다.
>

### **✔️인터넷에 엑세스 하기 위한 액세스 회선의 준비동작**

- ADSL이나 FTTH의 액세스 회선은 최초에 사용자명과 패스워드를 입력하여 로그인을 해야함. 그러나 BAS를 케이블로 고정적으로 접속하므로 PPP처럼 본인인지 확인할 필요가 없어진다.
- ADSL이나 FTTH는 PC에 글로벌 주소를 설정하지 않으면 인터넷에 접속할 수 없다.

### ✔️BAS의 기능

- BAS가 ADSL, FTTH 로그인 동작의 창구가 됨 (본인확인)
- 터널링이라는 개념을 사용하여 패킷을 운반하는 기능
- TCP/IP의 설정 정보를 통지 (PPP와 동일)

```text
터널링은 소켓과 소켓사이를 연결하는 TCP의 커넥션과 비슷하다. TCP 커넥션의 한쪽 출입구(소켓)에서
데이터를 넣으면 데이터가 그대로의 모습으로 반대쪽 출입구에 도착하는데 터널링은 이것과 같다. 
```

### ✔️터널링 기능에 의해 프로바이더에 패킷을 전달한다

- 사용자와 가장 가까운 전화국의 BAS와 프로바이더 라우터 사이에 있는 ADSL/FTTH 접속 서비스 사업자의 네트워크 안에 터널을 만든다. (액세스 회선이 프로바이더의 라우터까지 연결된다는 것)
- 터널링 실현 방식에는 TCP의 커넥션을 이용한 터널링과 캡슐화 방식의 터널링이 존재한다.
- 패킷을 그대로의 모습으로 운반할 수 있는 구조라면 원리적으로는 어떤 것에도 터널링을 이용할 수 있다.

## 프로바이더의 내부

> 액세스 회선을 통과한 패킷은 프로바이더의 라우터에 도착한다. 프로바이더의 라우터는 인터넷이 입구이며 여기에서부터 패킷은 인터넷의 내부로 들어간다. 인터넷의 실체는 단일 네트워크가 아닌 다수의 프로바이더의 네트워크를 서로 접속한 것이다. 프로바이더의 내부에는 POP 라우터와 NOC 라우터가 존재한다.
>

## 프로바이더를 경유하여 흐르는 패킷

### **✔️ BGP**

프로바이더 간에는 서로 경로 정보를 교환하여 라우터에 자동 등록한다. 예를 들면 프로바이더 A가 102.47.0.0 의 경로를 알고있으면 접속 상대 (프로바이더 B)에게 102.47.0.0로 가려면 여기로 경유하세요 라고 알리는 것이다. 프로바이더 B는 A에게 받은 정보를 경로표에 등록하고 목적지가 102.47.0.0 일 때 프로바이더 A에게 패킷을 전송한다. 이러한 구조를 **BGP**라고 한다.

### ✔️ **트랜지트(transit, peer)**

경로 정보 교환은 통지하는 경로 정보의 내용에 따라 두 가지 유형으로 나눌 수 있다. 하나는 인터넷의 경로를 전부 상대에게 통지하는 것이다.

<img width="340" alt="스크린샷 2022-05-05 오후 4 43 40" src="https://user-images.githubusercontent.com/56028408/166886633-f21e7632-7aee-43e0-8350-50cda065e094.png">

위 그림으로 예를 들자면, 프로바이더 E와 연결되어 있는 프로바이더 D가 프로바이더 E에 대한 경로를 프로바이더 B, C에게 전부 통지하는 것이다. 그러면 프로바이더 B,C도 연결되어 있는 다른 프로바이더에게 프로바이더 E의 경로를 통지하게 된다. 그 결과 프로바이더 D를 통과하여 인터넷의 모든 프로바이더에게 패킷을 보낼 수 있게되는데, 이것을 **트랜지트(transit)**라고 한다.

### **✔️비트랜지트(= peer)**

트랜지트와 또 다른 개념으로 비트랜지트 또는 피어라는 개념도 존재한다. 이것은 트랜지트와는 다르게 자신에게 직접 접속되어 있는 네트워크에 대한 경로만 상대에게 통지하는 것이다. 그러면 상대는 자신에게 직접 접속한 네트워크로 갈 패킷만 보내게 된다.

```text
💡 이더넷의 내부에는 BGP라는 구조를 사용하여 프로바이더간에 경로 정보를 교환한다. 
💡 BGP의 구조에서는 정보를 교환하는 상대를 'peer'라고 한다. 'peer'는 비트랜지터와 트랜지터를 모두 포함한다.
```

### **✔️ IX(Internet exchange)**

프로바이더끼리 접속하는 방법은 위 그림처럼 프로바이더 D와 프로바이더 C와 같이 1 대 1 형태로 접속하는 것이 기본이다. 지금도 이 방법을 사용 중이지만 이 방법만으로는 불편하다. 프로바이더끼리 1대1로 접속하는 방법밖에 없으면 모든 프로바이더가 서로 연결되어 있어야 한다. 현재는 국내에만 해도 여러 회사가 있으므로 이것은 보통일이 아니다. 이러한 상황에서 중심이 되는 설비를 설치하고 이것을 경유하여 접속하는 방법을 선택하면 통신 회선의 수를 억제할 수 있다. 이 경우 중심 설비를 IX라고 한다.

<img width="744" alt="스크린샷 2022-05-05 오후 5 21 23" src="https://user-images.githubusercontent.com/56028408/166886529-a204e1df-ccb3-42b6-aa06-f8972aadc274.png">


# Reference

성공과 실패를 결정하는 1%의 네트워크