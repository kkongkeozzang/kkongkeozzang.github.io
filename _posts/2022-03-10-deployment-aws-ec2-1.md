---
title: "AWS EC2(ubuntu)에 프로젝트 배포하기(2)"
use_math: true 
toc: true
toc_sticky: true
toc_label: 목차
categories:
  - deployment

tags:
  - 배포(Deployment)
  - AWS
  - EC2
  - RDS
  - JNDI
 

---



## 1. 이어서..  

이전 1편에는 AWS EC2 인스턴스에 톰캣을 설치하여 프로젝트를 두 개 배포하는 과정을 정리했다.  
DB 관련해서는 RDS 서버만 만들어두고 프로젝트에 연결을 시키지는 않았는데, 프로젝트에 DB를 연결하고 Jsp/Servlet과 Spring Framework는 어떻게 설정이 다른지 함께 정리하면서 배포 과정을 마무리 지으려고 한다.  
두 프로젝트가 하나의 프레임워크를 쓰거나 안쓰거나 였다면 쉬웠을 것 같은데, 달라서 조금 힘들었던 점이 있었다.  




## 2. 프로젝트에 DB 연결하기  



### 1) Jsp/Servlet 프로젝트  

첫 번째 프로젝트는 Jsp와 Servlet을 이용한 프로젝트이다.   
이때, DBCP를 자바가 아닌 톰캣이 가지고 있게 하기 위해 JNDI를 설정한다.  
자세한 내용은 [여기](https://kkongkeozzang.github.io/study-kh/servlet/jsp/study-54/#3-tomcat%EC%97%90-dbcp-%EC%84%A4%EC%A0%95-%EC%BD%94%EB%93%9C-%EC%A0%81%EC%9A%A9-jndi)를 참고하자.  



그래서 로컬에서와 동일하게 EC2 인스턴스 톰캣 폴더에 있는 `context.xml`에 DB 정보를 적어주었다.   

```xml
 <Resource
	name="jdbc/oracle"
	auth="Container"
	type="javax.sql.DataSource"
	driverClassName="oracle.jdbc.driver.OracleDriver"
	url="jdbc:oracle:thin:@엔드포인트:1521:ORCL"
	username="계정이름"
	password="계정비밀번호"
/>
```


### 2) Spring 프로젝트  

두 번째 프로젝트를 할 때는 JNDI 설정을 하지 않았다. 왜냐하면 Spring에서 기본적으로 관리가 가능하기 때문이다. 그래서 [여기](https://kkongkeozzang.github.io/study-kh/spring/study-78/#1-dbcp)에서처럼 JDBC와 DBCP는 설정해주었지만 JNDI 설정은 하지 않은 것이다.  
이처럼 간단한 개발을 할 때는 JNDI가 필요 없지만 웹 어플리케이션을 배포하여 운영할 경우에는 아아래와 같은 이유로 JNDI 설정을 해주는 것이 좋다.  

1. WAS 서버 담당자와 개발자가 다를 수 있기 때문  
2. 만약 동일한 DB를 쓰는 애플리케이션이 한 서버에 배포가 많이 된다면 각각 DB를 연결해야하는데 그렇게 되면 리소스가 낭비되기 때문  
3. DB 1에 장애가 생기면 백업 DB 2로 로드밸런싱을 해야하는데, 소스단에 DB 설정이 있으면 어렵다.  



나는 두 번째 프로젝트는 굳이 JNDI 설정을 해주지 않았다.

### 3) JNDI 설정하기  

1번에서처럼 첫 번째 프로젝트는 톰캣 서버에 직접 JNDI를 설정해주었고, 두 번째 프로젝트는 건들지 않았다.  
그리고 톰캣을 재실행했는데 아래와 같이 로그가 뜨고, 인스턴스를 재시작해도 동일했다.  

```
SEVERE [localhost-startStop-1] org.apache.catalina.core.NamingContextListener.lifecycleEvent Creation of the naming context failed:  
```

배포한 두 번째 프로젝트는 잘 되는데, 첫 번째 프로젝트에 DB와 연결된 기능을 들어가면   

```
javax.naming.NameNotFoundException: Name [jdbc/oracle] is not bound in this Context. Unable to find [jdbc].
```

이렇게 에러가 뜨면서 에러페이지로 넘어갔다.  
왜인지 모르겠지만 두 번째 프로젝트의 DB는 잘 가져오는데, 첫 번째 프로젝트의 DB는 찾지 못하는 것 같았다.   
그래서 첫 번째 프로젝트의 JNDI 설정을 바꿔주었다.  
혹시 위 1, 2 방법으로 했을 때 안되는 사람은 이 방법으로 진행하면 된다.  
(필자는 1번이 안되서 3번으로 성공했는데 블로그 쓰려고 정리하다가 1번 방법으로 바꿨는데도 잘 돌아간다.. 시간의 문제였나..? )  




[이곳](https://tnsgud.tistory.com/346)을 참고하여 설정했다.  
`server.xml`에  `<GlobalNamingResources>` 태그 안에 첫 번째 DB의 정보를 적어주었다.  
그리고 `context.xml`에 `<ResourceLink>` 태그를 주어 연결시켜주었다.  
이렇게하니 첫 번째 프로젝트와 두 번째 프로젝트 모두 에러 없이 잘 실행되었다.  

### 4) 결론  

결론적으로 Spring 프로젝트는 JNDI 설정을 따로 해주지 않았고, Jsp/Servlet 프로젝트는 JNDI 설정을 해주었다.   
처음 설정을 할 때는 JNDI가 뭔지, DB는 어떻게 연결하는지 DBCP는 뭐고 JDBC는 뭔지 헷갈려서 엄청 헤맸다. 어떻게 검색해야할지도 모르겠고 내가 지금 어떤 코드를 작성하고 있는지도 잘 몰랐다.  
그러나 과정을 정리하면서 헷갈렸던 개념도 조금씩 잡아가게 되었고, 두 프로젝트의 차이점도 이해할 수 있었다.  
헷갈렸던 DB 관련 개념들은 따로 포스팅을 하여 정리할 예정이다.  
이렇게 말도 많고 탈도 많던 프로젝트 두 개 배포하기 과정이 끝났다!  

## 3. 참고 사이트들  

<https://velog.io/@alsdn9501/AWS-EC2-Java-11-%EC%84%A4%EC%B9%98>  
<https://kitty-geno.tistory.com/26>  
<https://dlevelb.tistory.com/541>  
<https://forums.aws.amazon.com/thread.jspa?threadID=113068>  
<https://song8420.tistory.com/169>  
<https://huskdoll.tistory.com/575>  
<https://tttck88.tistory.com/47>  
<https://dingrr.com/blog/post/rds를-써야-하나요-ec2에-설치하면-안되나요>  
<https://velog.io/@namgonkim/m1-mac-AWS-RDS-%EC%98%A4%EB%9D%BC%ED%81%B4-%EC%83%9D%EC%84%B1%ED%95%98%EA%B3%A0-SQL-Developer%EB%A1%9C-%EC%A0%91%EC%86%8D%ED%95%98%EA%B8%B0>  
<https://velog.io/@dsunni/%EB%A1%9C%EC%BB%AC-Oracle-DB%EB%A5%BC-AWS-RDS%EB%A1%9C-%EC%98%AE%EA%B8%B0%EA%B8%B0>  
<https://websecurity.tistory.com/6>  
<https://www.kangtaeho.com/42>  
<https://tnsgud.tistory.com/346>    
