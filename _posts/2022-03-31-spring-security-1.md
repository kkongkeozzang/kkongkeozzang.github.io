---
title: "Spring Security 설정하기(1) (Spring MVC + Jsp)"
use_math: true 
toc: true
toc_sticky: true
toc_label: 목차
categories:
  - Spring Security
  - Spring
  - JSP

tags:
---


기존 Oracle DB가 연동 되어있는 실습 Spring Legacy Project 에 Spring Security 설정을 하는 과정을 정리해보려고 한다.  
기본적인 흐름은 [이곳](https://www.youtube.com/watch?v=AMsg4_-_fO4&list=PLGXpHMFOMTTbCC4t6WSoKfVnUxHmyGXKJ&index=12)을 따라서 하고 있지만, 완벽하게 들어맞지는 않으므로 그때 그때 찾아서 추가하는 방식..  
자세한 코드는 [여기](https://github.com/kkongkeozzang/TIL/tree/master/Spring_04)를 참조.  


## 1. 초기 설정

### 1) pom.xml 

pom.xml에 디펜던시 코드 추가

```xml
<!-- Spring Security -->
<!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-web -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-web</artifactId>
	<version>5.6.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-config -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-config</artifactId>
	<version>5.6.2</version>
</dependency>
```


### 2) web.xml

web.xml에 아래 filter 코드 추가  
filter name은 중요하다. 스프링 시큐리티 내부에 있는 것과 동일해야 하기 때문  
이로써 모든 요청이 필터를 거치게 됨.

```xml
<!--  Spring Security Filter -->
<filter>
     <filter-name>springSecurityFilterChain</filter-name>
     <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
	<filter-name>springSecurityFilterChain</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

### 3) security-context.xml


아래 `contextConfigLocation`에 `/WEB-INF/spring/security-context.xml`를 추가하기  
스프링 시큐리티는 단독으로 설정이 가능하기 때문에 기존의 root-context.xml과는 별도로 작성하는 것이 좋다.  
`contextConfigLocation`는 IoC 컨테이너 설정파일의 위치를 의미.  
security-context.xml을 적어줬기 때문에 생성해줘야한다.  

```xml
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>
      /WEB-INF/spring/root-context.xml
      /WEB-INF/spring/security-context.xml 
   </param-value>
</context-param>
```
security-context.xml 생성 후 Namespace에서 security 선택하고 다시 Source로 돌아오자. 

```xml
<security:http auto-config="true" use-expressions="false">
	<security:intercept-url pattern="/**" access="ROLE_USER"/>
</security:http>
```

기본 설정이 보통 true는 아니지만 여기서는 true로 할거다  
expression은 사용안함  
모든 요청에 대해 검사할 패턴 /**  
액세스 속성 : 어떤 유형인지 알려줌  

## 2. 인증 관리자 설정하기

id가 user이고 pw가 password인 유저는 ROLE_USER의 권한을 갖게 된다.  
추후 확장할 것임.  

```xml
<security:http auto-config="true" use-expressions="false">
	<security:intercept-url pattern="/**" access="ROLE_USER"/>
</security:http>

<security:authentication-manager>
	<security:authentication-provider>
		<security:user-service>
			<security:user name="user" password="password" authorities="ROLE_USER" />
		</security:user-service>
	</security:authentication-provider>
</security:authentication-manager>
```

## 3. 기본 시큐리티 로그인 설정 완료  

여기까지 끝내고 사이트에 다시 들어가면 스프링 시큐리티가 기본으로 제공하는 로그인 화면이 뜰 것이다.   
여기에 user / password를 넣으면   
500 에러가 뜨면서 There is no PasswordEncoder mapped for the id "null" 이 뜬다.  
스프링 시큐리티 4버전 이후부터는 모든 비밀번호는 암호화를 해야지만 사용이 가능하기 때문에, 비밀번호에 암호화를 사용하지 않겠다는 `{noop}`을 적어주면 제대로 로그인이 된다.  

```xml
<security:http auto-config="true" use-expressions="false">
	<security:intercept-url pattern="/**" access="ROLE_USER"/>
</security:http>

<security:authentication-manager>
	<security:authentication-provider>
		<security:user-service>
			<security:user name="user" password="{noop}password" authorities="ROLE_USER" />
		</security:user-service>
	</security:authentication-provider>
</security:authentication-manager>
```

pattern 속성에 access 가능한 url을 적어주고, access에 권한을 적어주면 마음대로 접근을 제한할 수 있다.  

그러나 (~~구린~~)기본 시큐리티 로그인 화면을 쓰는 사람은 없을 것이다.. 만들어놓은 이쁜 커스텀 로그인 화면을 시큐리티에 적용시켜야 한다.  

## 4. 커스텀 로그인 페이지 적용시키기  

일단 기본으로 로그인 페이지만 내가 만든 커스텀 페이지로 적용만 시킨다.  
우선 `security-context.xml`의 `http`코드에서 두 가지를 추가해주어야 한다.  
`csrf` 코드가 없으면 403 에러를 발생시킨다.   
기본적으로 스프링 시큐리티를 설정하게 되면 csrf 설정이 꼭 필요한데, 실습 정도는 그냥 disable로 비활성화 시켜주고 넘어가자.  
두 번째로는 `form-login` 코드인데,   
`login-page`속성은 login 페이지의 주소를 넣으면 된다.  
나의 경우는 예제가 member 폴더에서 login을 관리하므로 `/member/login`으로 써줬다.   
그리고 `login-processing-url`은 로그인 form 내에서 id와 pw를 넘기는 form 태그의 action 속성값을 직접 지정해준다.    

```xml
<security:http auto-config="true" use-expressions="false">
	<security:csrf disabled="true"/>
	<security:form-login login-page="/member/login" login-processing-url="/member/login"
		username-parameter="id" password-parameter="pw"/>
	<security:intercept-url pattern="/board/*" access="ROLE_USER"/>
</security:http>
```

여기까지 한다면 로그인 페이지로 이동했을 때, 내가 만든 로그인 jsp 페이지로 이동하는 것 까지 완료한 상태이다. 

## 5. 로그인 상태일 때, 비 로그인 상태일 때 구분  

home 페이지에 로그인 상태일 때와 비로그인 상태일 때를 달리 표현해보자.  
우선  jsp 맨 위에 security tag를 사용할 수 있도록 pom.xml에 라이브러리를 추가해주고,   

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-taglibs -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-taglibs</artifactId>
	<version>5.6.2</version>
</dependency>
```

`security-context.xml`에 `http`코드 위쪽으로 아래 코드를 적어준다.  
안적어도 동작하는데 문제 없긴 하지만 security 태그를 사용할 때 에러가 발생할 수도 있다고 한다.  

```xml
<bean class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler"/>
```

jsp 제일 위쪽에 아래 코드를 적어준다.  

```jsp
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>
```

그리고 choose문을 사용하여 인증되었을 때, ID를 출력해주고 환영해준다.  
또한 로그아웃과 회원 탈퇴도 여기에 적어준다.  
비인증 상태일 때는 로그인하기 버튼을 노출시켜준다.  

```jsp
<sec:authorize access="authenticated" var="authenticated"/>
<c:choose>
	<c:when test="${authenticated }">
		<sec:authentication property="name" var="id"/>
		${id }님 환영합니다.<br>
		<a href="/member/logout">로그아웃</a>
		<a href="/member/leave">회원탈퇴</a>
	</c:when>
	<c:otherwise>
		<a href="/member/toLogin">로그인하기</a>				
	</c:otherwise>
</c:choose>
```


아직 시큐리티와 DB 연동이 되지 않은 상태이므로, DB와 연결된 (특히 로그인 데이터) 기능을 사용하는 페이지로 이동하면 에러가 뜨는 상황이다.  
이후에는 (2) 포스팅에 계속~  





