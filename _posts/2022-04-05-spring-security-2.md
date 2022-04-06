---
title: "Spring Security 설정하기(2) (Spring MVC + Jsp)"
use_math: true 
toc: true
toc_sticky: true
toc_label: 목차
categories:
  - Spring Security
  - Spring
  - JSP

tags:
  - UserDetailsService
  - UserDetails

---

기존 Oracle DB가 연동 되어있는 실습 Spring Legacy Project 에 Spring Security 설정을 하는 과정을 정리해보려고 한다.  
기본적인 흐름은 [이곳](https://www.youtube.com/watch?v=AMsg4_-_fO4&list=PLGXpHMFOMTTbCC4t6WSoKfVnUxHmyGXKJ&index=12)을 따라서 하고 있지만, 완벽하게 들어맞지는 않으므로 그때 그때 찾아서 추가하는 방식..  
자세한 코드는 [여기](https://github.com/kkongkeozzang/TIL/tree/master/Spring_04)를 참조.  


## 1. 로그아웃하기

로그인과 마찬가지로 로그아웃도 post 방식으로 요청을 해야한다.  
그러나 처음에 csrf 설정을 비활성화 했으므로 이부분도 그냥 get 방식으로 요청하고 넘어가자.  

`security-context.xml`에 http 코드 사이 `intercept-url` 코드 위쪽에 적어준다.  
`logout-url`은 login 때와 마찬가지로 로그아웃할 url이며, `logout-success-url`은 말 그대로 로그아웃 후 넘어갈 url을 뜻한다.  

```xml
<security:logout logout-url="/member/logout" logout-success-url="/"/>
```


## 2. CustomUserDetailsService  

user 정보를 DB에 저장해서 사용하므로 DAO를 이용하여 user 정보를 받아와야 한다.  
UserDetailsService는 원하는 객체를 인증과 권한 체크에 활용할 수 있다.  


### 1) CustomUserDetailsService 클래스 생성  

새 패키지를 만들고 (kh.spring.security) 안에 class를 생성해준다.  
UserDetailsService 를 구현한다.  
loadUserByUsername 메소드를 오버라이드 하여 user의 정보를 가져온다.  

```java
package kh.spring.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import kh.spring.dao.MemberDAO;
import kh.spring.dto.MemberDTO;

@Component("customUserDetailsService")
public class CustomUserDetailsService implements UserDetailsService {
	
	@Autowired
	MemberDAO memberDAO;
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		MemberDTO user = memberDAO.selectById(username);
		
		if(user != null) {
			return new User(user.getId(), user.getPw(), AuthorityUtils.createAuthorityList(user.getRole()));
		}
		return null;
	}

}
```

### 2)SecurityUser 클래스 생성

loadUserByUsername 메소드의 리턴값을 보면 UserDetails로 되어있다. UserDetails는 조회한 사용자 정보를 담는 인터페이스이다.  
UserDetails를 구현한다.  
이미 MemberDTO가 있으므로 @Autowired 해서 가져와서 적당히 리턴값을 고쳐준다.  

```java
package kh.spring.security;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import kh.spring.dto.MemberDTO;


public class SecurityUser implements UserDetails {
	
	@Autowired
	private MemberDTO memberDTO;

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		List<GrantedAuthority> auth = new ArrayList<GrantedAuthority>();
		auth.add(new SimpleGrantedAuthority(memberDTO.getRole()));
		return auth;
	}

	@Override
	public String getPassword() {
		return memberDTO.getPw();
	}

	@Override
	public String getUsername() {
		return memberDTO.getId();
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}

}
```

### 3) security-context.xml 설정  

component-scan 코드를 적어준다.  

```xml
<context:component-scan base-package="kh.spring.security"/>
<context:component-scan base-package="kh.spring.dao"/> 
```


provider 설정을 바꿔준다.  

```xml
<security:authentication-manager>
	<security:authentication-provider user-service-ref="customUserDetailsService"/>
</security:authentication-manager>
```


### 4) 에러 해결  

#### NoSuchBeanDefinitionException  

```
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'customUserDetailsService': Unsatisfied dependency expressed through field 'memberDAO'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'kh.spring.dao.MemberDAO' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

MemberDAO 주입이 계속 실패했다.  
기존에 다른 기능들은 다 동작을 해서 뭐가 문제이지 하고 한참 들여다 봤는데 문제는 `CustomUserDetailService` 클래스에 MemberDAO를 Autowired해놓고 security-context.xml에 component-scan 코드를 넣어주지 않았다.  
(kh.spring.security 패키지는 component-scan 한 상황)  
MemberDAO는 스프링 컨테이너에서 scan 하기 때문에 따로 적어줄 필요가 없다고 생각했던 것이 오산이였다.  
`security-context.xml`에 dao 패키지를 component-scan 하는 코드를 적어주는 것으로 해결.  
참고  
<https://stackoverflow.com/questions/12334922/spring-and-spring-security-configuration-help-cannot-find-a-bean>  
<https://sjh836.tistory.com/165>  

#### 500 에러  

```
There is no PasswordEncoder mapped for the id "null"
```

아이디, 비번을 치고 로그인을 하니 500 에러가 뜨면서 위와 같은 에러가 떴다.  
암호화 설정을 안해줘서 나타난 에러로 1편에서 `{noop}` 으로 처리했었다.  
3편에서 암호화 작업을 들어가야곘다.  

















