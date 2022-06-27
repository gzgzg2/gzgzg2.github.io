---
layout: post
title: "[네트워크 스터디] Chapter_01 웹 브라우저가 메세지를 만든다"
categories:
  - networkStudy
tags:
  - HTTP
  - Socket
  - DNS
  - TCP
  - 성공과 실패를 위한 네트워크 1%의 원리

toc: true
toc_sticky: true

date: 2022-04-14
last_modified_at: 2022-04-14
comments: true
---

# 📗 Chapter_01 웹 브라우저가 메세지를 만든다
## **1. HTTP 리퀘스트 메세지를 작성한다**

### **1️⃣  URL**

**✔️ HTTP URL 의 구성**

ex ) [http://user:password@www.example.co.kr](http://user:passwork@www.cyber.co.kr):80/dir/file1.html

- 프로토콜 →  http
- 사용자명 (생략 가능) → user
- 패스워드 (생략 가능) → password
- 웹 서버의 도메인명 → [www.example.co.kr](http://www.example.co.kr)
- 포트번호 (생략 가능) → 80
- 파일의 경로명 → /dir/file1.html

**💡 HTTP 의 경우 기본 포트는 80이고 HTTPS의 기본 포트는 443이다.**

<br>

✔️ **URL 은 여러 종류가 있다.**


URL은 http로 구성되어있는 것 말고도 ftp: 나 mailto: 같은 다양한 프로토콜의 URL이 존재한다. 이유는, 브라우저는 웹 서버에 엑세스하는 클라이언트로만 사용하는 것이 아니라 파일을 다운로드/업로드하는 FTP의 기능이나 

메일클라이언트의 기능도 가지고 있기 때문이다. 브라우저는 URL을 토대로 여러개의 기능 중 어느것을 사용하여 데이터에 엑세스하면 될지를 판단한다. 

<br>

**✔️ 다양한 URL의 구성**

![image](https://user-images.githubusercontent.com/56028408/163106926-34b4cb16-6547-41bb-b016-0b2c4e6b081a.png)


이와같이 쓰는 방법은 다양하지만 모든 URL에는 하나의 공통점이 있다. URL 맨 앞에 있는 프로토콜에 따라서 액세스 하는 대상이 달라진다는 것이다. 

예를 들어 액세스 대상이 웹 서버라면 HTTP 프로토콜을 사용하고, FTP 서버라면 FTP, 메일을 송신할 때는 mailto 프로토콜을 사용하면 된다.

<br>


### 2️⃣  **브라우저의 URL 해독 방식**

웹 서버에 보내는 리퀘스트 메세지를 작성하기 전에 URL을 해독한다. 아래 내용은 브라우저가 웹 서버에 엑세스하는 경우를 예로들어 설명한다.

**URL을 요소별로 분리하여 나열한 것은 아래와 같다.**                                                                                             

```c
[http://www.example.co.kr](http://user:passwork@www.cyber.co.kr):80/dir/file1.html
[ http ]: + [ // ] + [ 웹 서버명 ] + [ / ] + [ 디렉토리명 ] +[ / ] + .... + [ 파일명 ]   
```

http는 앞서 설명한 프로토콜이 되는 것이고 //는 나중에 이어지는 문자열이 서버의 이름임을 나타낸다.

웹 서버명 다음에 이어지는 것은 서버에 파일이 저장된 위치인데 이는 생략이 가능하다. 

 ****

**✔️  파일명을 생략한 경우** 

ex) [http://www.example.co.kr](http://user:passwork@www.cyber.co.kr)

예시 url은 파일명을 생략했을 때의 url이다. 파일명을 생략하면 서버가 어느 파일에 액세스해야 하는 지 알 수 없다. 

그래서 이와같은 문제를 대비하여 서버에서 파일명이 생략되었을 때 액세스할 default 파일을 설정해둔다. 대부분의 서버는 index.html 이나 default.html 이라는 파일명으로 설정해둔다.

**💡  브라우저가 가장 먼저 하는 일은 URL 해독이다.**

### 3️⃣ HTTP의 기본 개념

> Hyper-Text Transfer Protocol(HTTP)란 지정된 규약을 지켜 웹 서버와 데이터를 주고받는 서버/클라이언트 모델을 따르는 비연결성 & 무상태 프로토콜이다.
>
> 애플리케이션 레벨의 프로토콜로 TCP/IP 위에서 작동한다. HTTP 패킷에는 여러 메타데이터와 성공 실패 정보가 담겨있으므로 서버와 클라이언트가 원활하게 대화할 수 있다.
> 

**💡  비연결성**

연결을 유지하지 않는 것  특정한 옵션을 주면 일정시간동안 연결을 유지할 수는 있다.

**💡  무상태** 

서버가 클라이언트의 상태를 보존하지 않는 것 매번 요청에 모든 상태값들을 전달해줘야 함 상태를 유지해줘야 할 경우 쿠키나 세션을 사용 함


<br>


**✔️ HTTP Reuqest 패킷 구성**

![image](https://user-images.githubusercontent.com/56028408/163209206-2d7e3008-230e-4da7-9041-453bcfa2d23d.png)


HTTP 요청 패킷은 크게 요청라인, 메세지 정보, 본문으로 나눌 수 있다. 

- 요청라인 → HTTP Method, URL, HTTP 버전
- 메세지 정보 → 해당 요청에 대한 메타데이터를 담고있는 공간
- 본문(body) → 해당 요청의 실제 내용. POST가 아니여도 body를 사용할 수 있지만, 주로 Body를 사용하는 메소드는 POST이다.


<br>


**✔️ HTTP Response 패킷 구성**

![image](https://user-images.githubusercontent.com/56028408/163209182-d49ace5d-d8c1-4e7b-aa31-976c9cdbafb5.png)

HTTP 응답 패킷은 크게 상태라인, 메세지 정보, 본문으로 나눌 수 있다. 

- 상태라인 → HTTP Method, 버전, 응답코드
    - 응답코드는 주로 권한과 서버에러, 클라이언트에러 등으로 나눠서 성공과 실패 여부를 판단하는데에 사용된다,
- 메세지 정보 → 해당 응답에 대한 메타데이터를 담고있는 공간
- 본문(body) → 요청의 Body와 일반적으로 동일하다. body 존재할 수도 있고 존재하지 않을수도 있다. 가장 많이 사용되는 Body의 데이터 타입은 JSON(JavaScript Object Notation)이다.

<br>

**✔️ HTTP Method**

HTTP 메소드를 정리한 표이다. 버전마다 지원되는 메소드가 다르다.

| 메소드 | 의미 |
| --- | --- |
| GET | 지정한 정보를 도출할 때 사용된다. 파일의 경우 해당 파일의 내용을 되돌려보낸다. |
| POST | 클라이언트에서 데이터를 body에 담아 송신할 때 사용된다. |
| PUT | URI로 지정한 서버의 파일을 치환한다. 파일이 없는 경우 새로 등록한다. |
| DELETE | URL로 지정한 서버의 파일을 삭제한다. |
| PATCH | 리소스의 일부분만을 수정하는데에 쓰인다. |
| OPTION | 통신 옵션을 통지하거나 조사할 때 사용된다. |
| HEAD | GET과 거의 같지만 데이터의 내용을 돌려보내지 않고 HTTP 메세지 헤더만을 반송한다. 속성 정보를 조사할 때 사용된다. |
| TRACE  | 서버측에서 받은 리퀘스트 라인과 헤더를 그대로 클라이언트에 반송한다. 프록시 서버 등을 사용하는 환경에서 리퀘스트가 치환되는 상태를 조사할 떄 사용된다. |
| CONNECT | 암호화한 메세지를 프록시로 전송할 때 이용된다. |

<br>

**✔️ HTTP 간단한 통신과정 예시**

![image](https://user-images.githubusercontent.com/56028408/163209168-f37b8432-03fc-4dbf-883e-001f24d63871.png)

1. 웹서버에 특정 리소스를 얻기 위해 URI와 헤더에 요청에 필요한 정보를 담아 서버에 전송한다. 
2. 해당 리소스가 존재하면 응답 헤더에 필요한 정보를 담아서 보내거나 바디에 해당 요청 데이터를 전송한다.

🌟 **실제로 요청과 응답 사이에 검증절차나 여러가지 일들이 일어나지만 자세한 내용은 생략하겠다.**

## 2. 웹 서버의 IP 주소를 DNS 서버에 조회한다

> HTTP 메세지를 만드는 것에 성공하면 이것을 OS에 의뢰하여 액세스 대상의 웹 서버에게 송신한다. 
>
> 브라우저는 URL을 해독하거나 HTTP 메세지를 만들 수는 있지만 메세지를 네트워크에 송출하는 기능은 없으므로 OS에 의뢰하는 것이다. 
>
> 이때 URL 안에 쓰여있는 서버의 도메인 명에서 IP 주소를 조사해야하는데, 이는 OS에서 송신을 의뢰할때는 도메인명이 아니라 IP 주소로 메세지를 받을 상대를 지정해야하기 때문이다.
> 

**✔️  도메인명과 IP 주소를  구분하여 사용하는 이유**

TCP/IP 네트워크는 IP 주소로 통신 상대를 지정하므로 IP주소를 모르는 상대에게 메세지를 전달할 수 없다. 이러한 이유 때문에 도메인 명과 IP 주소를 구분하는 것이다. 웹 브라우저는 Socket 라이브러리를 사용하여 도메인 명으로 IP 주소를 조회한다. 
그렇다면 URL 안에서는 서버명이 아니라 IP 주소를 사용하는 것이 더 효율적이라는 생각이 들 수 있다. 실제로 서버명 대신에 IP 주소를 사용해도 올바르게 작동한다.그렇지만 기억하기 어렵다는 단점이 있다. 실행 효율 관점에서 바라보면 IP 주소대신 도메인명으로 사용하는 것이 좋은 방법이라고 할 수 없다. IP 주소는 32비트로 4바이트에 해당하는 개수밖에 없지만, 도메인 명은 수십 바이트 부터 최대 255 바이트 까지 존재한다. 그러면 그만큼 라우터가 부하되어 데이터를 운반하는 동작에 더 많은 시간이 걸릴 것이다. 하지만 이름을 알면 IP 주소를 알 수 있다거나 IP 주소를 사용하면 이름을 알 수있다는 원리를 이용하여 양쪽의 차이를 해소한 DNS가 라우터 부하의 문제점을 해결해준다.

<br>

**✔️ Socket 라이브러리의 IP 주소 조회**

웹 브라우저는 Socket 라이브러리의 도움을 받아 DNS서버에서 도메인명의 IP 주소를 조회할 수 있다.

DNS 서버에서 조회한다는 것은 DNS 서버에 메세지를 보내고 반송되는 응답 메세지를 받는 것이다. 이것은 DNS 클라이언트로 동작한다고 말할 수 있다. DNS 클라이언트에 해당되는 것을 DNS 리졸버(=리졸버)라고 한다.

DNS 원리를 이용하여 IP 주소를 조사하는 것을 네임 리졸루션 (name resolution) 이라고 하는데 리졸루션을 수행하는 것이 리졸버이다. 

<br>

**💡 Socket 라이브러리**

- OS를 이용하여 네트워크의 기능을 호출하기 위한 프로그램의 부품집

<br>

### ✔️ 리졸버 내부의 작동

![image](https://user-images.githubusercontent.com/56028408/163209513-02792b34-4c07-4427-9ed5-0f0a68b403ce.png)

✔️  **리졸버를 이용하여 DNS 서버를 조회한다.**

리졸버의 프로그램명(gethostbyname)과 웹 서버의 이름을 쓰기만 하면 리졸버를 호출할 수 있다. 

1. 브라우저 애플리케이션이 Socket 라이브러리의 리졸버를 호출한다. 이때 gethostbyname에 웹서버 이름을 전달한다.
2. 리졸버가 DNS 서버에 조회 메세지를 보낸다. 메세지 송신 동작은 OS 내부의 프로토콜 스택을 호출하여 수행한다.
3. DNS 서버에서 돌아온 응답 메세지를 수신한다.
4. 응답 메세지에서 IP 주소를 추출하고 메모리 영역에 저장한 뒤 애플리케이션으로 돌아간다.

<br>

🌟  DNS 서버에 메세지를 송신할 때도 DNS 서버의 IP 주소가 필요한데, 이는 컴퓨터의 TCP/IP 설정 항목의 하나로 컴퓨터에 미리 설정되어 있다. 

## 3. 전 세계의 DNS 서버가 연대한다

### 1️⃣ **DNS 서버의 기본동작**

DNS 서버의 기본 동작은 클라이언트에서 조회 메세지를 받고 조회의 내용에 응답하는 형태로 정보를 회답하는 일이다. 

<br>

**✔️  조회 메세지 구성**

- 이름
    - 서버의 메일 배송 목적지와 같은 이름, 도메인 명
- 클래스
    - 네트워크 종류를 검토하기위해 사용되는 것 현재는 인터넷 외의 네트워크는 소멸되었기 때문에 항상 인터넷을 나타내는 IN 값이 전달된다
- 타입
    - 이름에 어떤 타입의 정보가 지원되는지를 나타낸다. 타입이 A이면 이름에 IP 주소가 지원되는 것을 나타내고 MX이면 이름에 메일 배송 목적지가 지원된다는 것을 나타낸다. 타입에 따라 클라리언트에 회답하는 정보의 내용이 달라진다.

이름이 [www.example.com](http://www.example.com) 인 서버의 IP 주소를 조사할 때 클라이언트는 다음과 같은 메세지를 보낸다. 

해당 메세지를 수신했을 때 DNS가 어떠한 동작을 하는지 알아보자

<br>

**[요청 메세지 예시 ]**

- 이름 = www.example.com
- 클래스 = IN
- 타입 = A

<br>

**[DNS 서버 내부 등록 정보 예시]**

| 이름 | 클래스 | 타입  | 클라이언트에 회답하는 항목 |
| --- | --- | --- | --- |
| www.example.com | IN | A | 192.168.2.5 |
| example.com | IN | MX | 192.168.2.6 |

<br>

**[DNS 서버의 동작]**

1. DNS가 메세지를 수신하면 DNS 서버는 이름, 클래스, 타입의 값이 일치하는 데이터를 찾는다. 
2. 일치하는 데이터가 있으면 요청 값에 맞는 데이터를 클라이언트에게 전송한다. (예제를 기준으로 하면 192.168.2.5) 

 **💡  DNS 서버는 서버에 등록된 도메인명과 IP 주소의 대응표를 조사하여 IP 주소를 회답한다.**

<br>

### 2️⃣  도메인의 계층

> 인터넷에선 막대한 수의 서버가 존재하기 때문에 모든 도메인 명을 하나의 DNS 서버에 등록하는 것은 불가능하다. 
>
> 이러한 문제점은 정보를 분산시켜 다수의 DNS 서버에 등록하는 방식으로 해결할 수 있다. 다수의 DNS 서버는 계층적으로 구성되어있는데, 이는 조회 메세지를 수신한 DNS 서버에 해당 정보가 등록되어 있지 않아도 계층적으로 연대하고 있는 다른 DNS 서버에서 찾을 수 있다.
> 

DNS 서버에 등록된 모든 정보는 모든 도메인명이라는 계층적 구조를 가진 이름이 붙여져 있다. DNS에서 취급하는 이름은 [www.example.com](http://www.example.com) 처럼 점으로 구분되어 있는데, 점으로 게층을 구분한다. 오른쪽에 위치한 것이 상위 계층이다. 계층구조와 같은 형태로 DNS 서버가 배치되며,  각 계층은 각자의 DNS 서버에 저장된다.

<br>

**[계층 구조 예시]**

URL → [www.example.com](http://www.example.com) 

오른쪽에 위치한 것이 상위 계층이다. 

- com
    - example
        - www

<br>

**✔️  담당 DNS 서버를 찾아 IP 주소를 가져온다** 

> 인터넷에는 DNS 서버가 수만 대가 있으므로 닥치는 대로 이를 뒤지면서 해당 도메인의 IP를 찾을 수는 없다.
> 
> 
> 그래서 다음과 같은 방법으로 이를 해결한다. 
> 

#### **[예시]**

URL → [www.example.com](http://www.example.com) 

예시 URL 기준으로 최상위 도메인은 com이다. 하지만 실제로 URL은 com이나 kr 위에 루트 도메인이 존재한다. 루트 도메인은 com이나 kr같은 도메인 명이 존재하지 않기 때문에 보통 도메인을 쓸 때는 이것을 생략한다. 명시적으로   [www.example.com](http://www.example.com). 처럼 URL 끝에 마침표를 찍기도 하지만 보통은 그렇게 사용하지 않는다. 

1. 하위의 도메인을 담당하는 DNS 서버의 IP 주소를 상위의 DNS 서버에 등록한다. 
2. 상위의 DNS 서버를 또 그 상위의 DNS 서버에 등록한다. 
3. 상위의 DNS 서버가 존재하면 이를 반복한다.

💡 **이러한 방식은 상위 DNS 서버에서 하위의 DNS 서버의 IP 주소를 알 수 있다. (루트 도메인을 모든 DNS 서버에 등록하면 루트 도메인의 모든 하위 DNS 서버를 조회할 수 있음)** 

<br>

#### **[IP 조회]**

![image](https://user-images.githubusercontent.com/56028408/163209731-4d525bc3-91f4-4d27-8a0e-e297b3456a1a.png)


1. 가장 가까운 DNS 서버에게 해당 도메인 명의 IP를 요청한다.
2. IP가 등록되어있지 않을 경우 루트 도메인 서버에게 해당 IP를 요청한다.
3. 루트 도메인에도 해당 IP가 등록되지 않았을 경우 하향식으로 접근하여 IP를 찾을 때까지 하위 도메인 서버에 해당 IP 정보를 요청한다. 

<br>

💡  **앞서 설명한 내용은 기본이 되는 동작을 설명한 것으로 현실의 인터넷과 모든 동작이 일치하진 않는다. 현실의 인터넷은 한 대의 DNS 서버에 복수의 DNS 서버를 등록할 수 있으므로 한 도메인에 한대씩 DNS 서버가 존재한다고 단정할 수 없다.**

### 3️⃣  DNS 서버는 캐시 기능으로 빠르게 회답할 수 있다.

DNS 서버는 한번 조사한 도메인 명을 캐싱할 수 있다. 요청받은 도메인 명이 캐싱되어 있으면 캐싱되어 있는 정보를 전달한다. 정보가 없을 경우에도 캐싱하기 때문에 정보가 존재하지 않는 다는 것도 빠르게 회답할 수 있다. 하지만 캐시된 정보가 변경될 수도 있으므로 캐시안에 저장된 정보를 올바르다고 단언할 수는 없다.

## 4. **프로토콜 스택에 메시지 송신을 의뢰한다**

IP 주소를 조사를 마치면 액세스 대상 웹 서버에 메세지를 송신하도록 OS 내부 프로토콜 스택에 의뢰한다.

이 동작에서도 Socket 라이브러리를 사용하는데, 복수의 부품을 결정된 순서대로 호출해야하므로 복잡하다.

<br>

🌟 **Socket 라이브러리를 이용한 데이터 송 수신 동작**

![image](https://user-images.githubusercontent.com/56028408/163209131-ddef6b53-6743-413a-abae-c4b862dcfde6.png)

1. 서버측에서 소켓을 생성하고 클라이언트가 파이프를 연결하기를 기다린다. (소켓 생성 단계)
2. 클라이언트가 서버측 소켓에 파이프를 연결한다 (소켓 접속 단계)
3. 연결이 성공하면 데이터를 송 수신 한다 (송 수신 단계)
4. 송 수신 동작이 끝나면 파이프 연결을 해제한다. (연결은 클라이언트, 서버 둘 중 어느 쪽에서 분리해도 상관없다.)

<br>

**🌟  결정된 순서대로 Socket 라이브러리를 호출하는 애플리케이션 예시**

![image](https://user-images.githubusercontent.com/56028408/163209107-a08c09a6-c0f1-4509-b64c-adae70159364.png)

- socket()
    - 소켓을 생성하고 완료되면 제어권을 애플리케이션에게 넘긴다. 소켓이 생성되면 **디스크립터**가 반환되는데, 이는 소켓을 식별하기 위한 식별자의 용도로 사용된다.
- connect()
    - 디스크립터, 서버의 IP 주소, 포트번호 세 가지 값을 입력하여 특정 소켓에 연결을 요청한다.
- wirte()
    - 송신 데이터를 메모리에 할당하고 wirte() 함수를 호출할 때 디스크립터와 송신데이터를 지정한다. 소켓에는 연결된 상대가 기록되어 있으므로 디스크립터로 소켓을 지정하면 연결된 상대가 판명된다.
- read()
    - 메세지를 수신할 때 사용된다. 수신한 응답메세지는 수신 버퍼에 저장된다. 수신버퍼는 애플리케이션 내부에 마련된 메모리 영역이므로 수신 버퍼에 메세지를 저장한 시점에 애플리케이션에게 메세지를 넘겨준다.
- close()
    - 메세지 송 수신이 완료되면 연결을 종료하는 함수 만약 read 동작을 수행하고 있는 도중에 연결이 종료되면 수신한 데이터를 건네주고 연결을 종료한다.

<br>

**🌟  디스크립터  =  애플리케이션이 소켓을 식별하는 용도**

**🌟  IP와 포트번호  = 클라이언트와 서버 간에 상대의 소켓을 식별하는 용도**

**🌟  요청에는 하나의 리소스 정보만을 표시하고 있기 때문에 여러 파일의 정보를 읽으려면 요청을 각각 보내야한다. 요청을 각각 보내면 매번 TCP Connection 을 새로 맺어야하는데, 이는 비효율적일 수 있다. 그래서 HTTP 1.1 버전부터 일정시간동안 연결을 유지할 수 있는 옵션이 추가되었다.**

<br>
<br>

# Reference

[https://velog.io/@anhesu11/HTTP-기본-이론-정리](https://velog.io/@anhesu11/HTTP-%EA%B8%B0%EB%B3%B8-%EC%9D%B4%EB%A1%A0-%EC%A0%95%EB%A6%AC)

성공과 실패를 결정하는 1%의 네트워크