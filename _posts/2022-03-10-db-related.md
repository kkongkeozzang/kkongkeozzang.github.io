---
title: "JDBC, DBCP, JNDI... 헷갈리는 개념 정리"
use_math: true 
toc: true
toc_sticky: true
toc_label: 목차
categories:
  - DB

tags:
  - DataSource
  - Connection pool
  - DBCP (DataBase Connection Pool) Library
  - JDBC
  - Spring JDBC
  - JNDI

---



## 1. 개념 정리   

AWS EC2 인스턴스에 프로젝트를 배포하면서, DB 관련된 용어들이 헷갈리면서 개념 정리가 필요하다고 생각했다.  
아래는 DB 연동하며 자주 보았던 키워드들을 정리한다.  



## 2. DB 관련 자주 보았던 키워드  


### 1) DataSource  

데이터베이스에 접속하기 위한 모든 정보를 해당 컨텍스트에 하나의 이름으로 등록한 후 그 정보를 객체화한 클래스이다.  
즉, 데이터베이스에 접속하기 위한 모든 정보와 Connection pool을 이용한 Connection을 관리하는 객체이다.  

DataSource 설정은 [Spring JDBC](https://kkongkeozzang.github.io/study-kh/spring/study-79/#6-spring-jdbc), 트랜잭션 처리, JPA 연동 등 다양하게 사용된다.
DataSource가 있기 때문에 JNDI를 사용할 때 이름을 매칭하여 쉽게 DB 정보들을 가져올 수 있다.  


### 2) Connection pool  

> Connection pooling is a technique of creating and managing a pool of connections that are ready for use by any thread that needs them. Connection pooling can greatly increase the performance of your Java application, while reducing overall resource usage.  
출처 : [docs.oracle.com](https://docs.oracle.com/cd/E17952_01/connector-j-8.0-en/connector-j-usagenotes-j2ee-concepts-connection-pooling.html)  

정리하자면 Connection을 관리하는 기술을 뜻한다. 울타리를 생각하면 쉬운데, Connection을 가지고 있는 울타리를 만들어 Connection을 주고 다 쓴 Connection을 다시 가져오며 전반적인 리소스 사용을 줄일 수 있다.  
DBMS는 동시에 사용할 수 있는 Connection을 제한하기 때문이다.  
이전 포스팅에 쉽게 정리한 내용이 있다. [Connection Pool이란](https://kkongkeozzang.github.io/study-kh/network/jdbc/study-29/#4-connection-pool-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0)  



### 3) JDBC  

> 데이터베이스에 연결 및 작업을 하기 위한 자바 표준 인터페이스.  
Java와 DBMS가 서로 통신할 수 있게끔 만들어주는 라이브러리.  
(Oracle) Java DataBase Connectibity  

이전에 [수업 정리](https://kkongkeozzang.github.io/study-kh/java/jdbc/study-22/#1-jdbcojdbc%EB%9E%80)한 것을 참고했다.  
왜 Oracle을 다운받으면 JDBC 라이브러리가 같이 받아질까? 왜 다른 언어가 아닌 자바 라이브러리를????   
이 생각이 계속 들어 검색을 해봤는데, 아무리 해도 안나왔다.  
결론은 이상한데에서 나왔다. 오라클이 썬 마이크로시스템즈를 인수하면서 Java의 저작권을 소유했기 때문...(이라고 생각한다.)  
어쨌든 이런 자바 API를 가지고 구글과 오라클이 소송까지 갔던 것을 보니 비슷한 맥락이지 않을까 싶다. 오라클 쓰세요~ 쓰시는 김에 자바 쓰시면 이 라이브러리 쓰셔도 되고~ 이런 느낌이 아닐까..(확실하지않음)  



어쨌든 JDBC는 DB와 자바를 이어주는 `어댑터`같은 역할을 한다고 보면 된다. Oracle을 쓰다 다른 DBMS로 바꾸려고 할 때, JDBC를 사용한다면 전체 코드를 뒤집어 엎을 필요가 없다.  





### 4) DBCP  

Connection pool을 잘 사용하고 관리하는 것이 중요하고 핵심적인 부분이기 때문에 사람들은 편하게 쓰기 위해 라이브러리를 만들었다.  
이 라이브러리를 `DBCP`라고 부른다.  
간단하게 생각하면 Connection pool을 잘 관리할 수 있게 하는 라이브러리이다.  


### 5) JNDI  

자원에 이름과 속성을 부여하여 서비스를 사용할 수 있게 하는 API
디렉터리 서비스에서 제공하는 데이터 및 객체를 발견하고 참고하기 위한 자바 API  
쉽게 말하면 외부에 있는 객체를 가져오기 위한 기술



JNDI를 사용하는 이유는 프로젝트 내부에 DB 정보를 담아두는 것이 아니라 Tomcat과 같은 WAS 서버에 DB 정보를 담아두고 사용하기 위해서이다.  