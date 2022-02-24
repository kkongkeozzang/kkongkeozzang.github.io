---
title: "Jar & War 그리고 배포(Deployment)"
use_math: true 
toc: true
toc_sticky: true
toc_label: 목차
categories:
  - deployment

tags:
  - 배포(Deployment)
  - jar
  - war
  - AWS
  - WAS

---



포트폴리오를 준비하는 과정에서, 프로젝트를 배포해보는 경험이 필요하다고 판단했고, 요 며칠 AWS를 이용한 배포 과정을 공부하면서 기본 개념이 부족한 것 같아 배포에 대한 개념부터 다시 정리한다.  


## 1. 배포(Deployment)란? 

개발이라는 것은 결과적으로 우리가 소스코드를 작성을 하고 그것을 컴파일하여 바이너리 파일로 만드는 작업이다.   
이렇게 만든 바이너리 파일(실행파일)이 Native 어플리케이션이라고 한다면 jar 라는 파일로 만들어진다. 그리고 이 파일을 다른사람에게 보낸다(건네준다, Disket / CD으로 판다 등) 라는 행동이 `배포`라는 과정이 된다.  
참고로 DIsket과 CD는 불법복제가 쉬워 native 시장의 몰락의 주범이 되었다..  



웹 어플리케이션을 만드는 과정도 동일하다. 소스코드를 작성하고 컴파일해서 실행파일을 만드는 것까지는 동일하지만, jar가 아닌 war 파일로 만들어진다.  
그러나 배포 과정이 다른데, war파일을 직접 건네주지 않는다.  대신 웹서버(또는 웹어플리케이션서버)에 배포하게된다(사용자들에게 주지 않고 사용자들이 접속할 수 있는 서버에 올린다)  
이로써 불법복제 X 추가적인 비용 X 가 되면서 웹앱 시장이 빠르게 성장하게 된다.  



웹 어플리케이션을 웹서버(또는 웹어플리케이션서버)에 배포할 때 jar 또는 war 파일을 만들어서 웹서버에 업로드하면 다른 사용자들도 특정 주소를 입력하여 웹 어플리케이션을 이용할 수 있게 된다.  이때 집에 있는 컴퓨터를 서버로 사용하면 24시간 컴퓨터를 켜놔야 하므로 큰 기업에서 빌려주는 서버를 가져다가 쓰는게 대부분이다.(예를 들면 AWS)  



## 2. JAR 와 WAR?  

여러 사이트들을 돌아 정리했지만 번역은 실력이 떨어져 그냥 원문을 읽는게 도움이 된다..  

### 1) JAR (Java Archive)  


> A JAR file is a file that contains all components to make a self-contained executable Java application.


직역하자면 Java 보관소로, 자바 클래스 파일 등을 모아둔 압축 파일이라고 생각하면 된다.  
자바 환경 위에서 바로 동작할 수 있다.  
자바 어플리케이션을 실행시키기 위한 모든 구성품을 포함하는 파일이다.  

### 2) WAR (Web Application Archive)  

> A WAR file contains files related to a web project.


웹 어플리케이션 보관소.  
JAR 파일, JSP, Servlet, XML, HTML등등으로 구성되어있다.  
웹 어플리케이션 구성요소들을 모아놓은 파일들로 어플리케이션 서버가 있어야 동작한다.  


### 3) 차이점?  

war도 쉽게 말하면 jar 파일이라고 할 수 있다. 다만 war 파일은 오직 웹 어플리케이션을 위해 사용되는 파일이다.  
war가 조금 더 큰 범위의 확장자? 라고 생각하면 될 듯 하다.  


### 4) 참고 사이트  

[https://pediaa.com/what-is-the-difference-between-jar-and-war-files/](https://pediaa.com/what-is-the-difference-between-jar-and-war-files/)
[https://stackoverflow.com/questions/5871053/difference-between-jar-and-war-in-java](https://stackoverflow.com/questions/5871053/difference-between-jar-and-war-in-java)
