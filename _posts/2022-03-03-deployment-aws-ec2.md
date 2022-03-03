---
title: "AWS EC2(ubuntu)에 프로젝트 배포하기"
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
  - ubuntu
  - putty
  - puttygen
  - RDS
  - filezilla
  - Linux

---

학원 교육을 받을 때는 AWS 담당자를 따로 두고 조원 중 한명이 AWS를 담당하여 배포하는 식으로 했다. 나는 AWS까지 담당하기에는 벅차다고 생각했기에 나중에 수료후 해야겠다고 다짐했다.  
프로젝트 한개를 AWS EC2 Window 서버에 배포하는 것은 조금만 찾아봐도 자료가 많이 나오는데, 내가 했던 방법은 잘 나오지 않아 따로 정리해야겠다는 생각이 들었다.  
아래는 그 과정이다.  
검색하면 잘 나오는 부분들은 따로 정리하지 않았다.  


## 1. 내 상황..  

보통 세미 프로젝트인 Jsp/Servlet 프로젝트를 서버에서 내리고 파이널 프로젝트를 배포하여 포트폴리오를 만든다.  
그런데 나는 세미 때 했던 프로젝트도 버리고 싶지 않아서 두 개를 모두 배포하고 싶었는데, AWS EC2 인스턴스를 각각 생성하여 탄력적 IP를 두개 두기에는 과금이 무서웠다.(탄력적 IP는 프리티어일 때 1개이면 무료)  
검색했지만 내 상황과 비슷한 게 없어서 여기저기 검색해서 짜깁기해서 성공했다! 혹시 나와 비슷한 상황인 분들이 있다면 아래 정리된 순서를 따라가면 어렵지 않게 성공할 수 있을 것이다..  
오래걸렸지만 이번 기회로 `AWS`, `WAS`, `TOMCAT`, `LINUX`, `파일경로`에 대해 조금 더 알게되었다.  

- Jsp/Servlet 프로젝트 (Dynamic Web Project) 1개  
-  Spring MVC 프로젝트 (Spring Legacy Project) 1개  
-  프로젝트 두 개를 한 AWS EC2 인스턴스에 올리고 싶음(각각 url에서 작동할 수 있도록)  
-  DB는 기존에 쓰던 것을 백업하여 옮겨야 함  
-  DB도 두개임  
-  파일 업로드&다운로드는 외부 폴더를 설정  
-  AWS EC2 인스턴스는 ubuntu 사용  
-  Java11, Tomcat8.5, Oracle11g 사용했음 (프로젝트에서)  

결과적으로 아래와 같은 환경으로 배포에 성공했다.  

-  [3.38.240.8:8888](http://3.38.240.8:8888/)으로 1번 프로젝트 배포  
-  [3.38.240.8:9999](http://3.38.240.8:9999/)으로 2번 프로젝트 배포  
-  AWS EC2 인스턴스 1개 사용 + 탄력적 IP 적용  
-  DB는 Oracle 12 v26 RDS 서버 사용  
-  계정을 각각 생성하여 DBA 권한 줘서 각 프로젝트에 연결  
-  외부 파일 경로 설정(추후 S3로 옮겨볼 생각..)  
-  ubuntu 서버 + RDS 서버 사용  
-  Oracle 빼고 버전은 같게 설정해주었다.  



## 2. 배포 과정  



### 1) AWS EC2 인스턴스 생성 및 접속 

계정만들기는 기본 중에 기본. 검색하면 수두룩하게 나오니 알아서 따라하자.  
`ubuntu 20.04`로 인스턴스 생성하고, 탄력적 IP(고정IP) 설정해준다.  
과금 알람 설정은 꼭 해주기!  
`putty`와 `puttygen`을 다운받아 key를 생성해주고 접속 후 실행한다.  
`putty`는 내 로컬에서 원격으로 서버에 접속할 수 있게 도와주는 프로그램이다.  


### 2) 자바 & 톰캣 설치   

자바11을 설치한다. [이곳](https://velog.io/@alsdn9501/AWS-EC2-Java-11-%EC%84%A4%EC%B9%98)의 도움을 받았다.  
톰캣 8.5를 설치한다. [이곳](https://kitty-geno.tistory.com/26)의 도움을 받았다.  
톰캣 실행&종료 커맨드는 잘 적어두자. 계속 사용할 것이기 때문  
다만 마지막 환경변수 설정을 하니 실행이 안되서, [이곳](https://dlevelb.tistory.com/541)에 적혀있는 것처럼 환경변수를 돌려주었고, 재설치(여러번의 시행착오가 있었다) 후에는 환경변수 설정은 건너뛰었다.  
Oracle은 RDS 서버에 두기 때문에 패스~  



### 3) 인바운드 규칙 설정  

사용하는 포트들을 열어주는 과정이다.  
`8888`, `9999`, `80`, `22`, `443`, `1521`을 열어주었다.  
지금 생각해보니 `80`을 열 필요는 없는 것 같다. 어차피 `8888`과 `9999`를 쓰니까..  
아웃바운드는 전체를 대상으로 열어준다.  
이대로 적용한 뒤, `자신의 IP:포트번호`로 접속하면 톰캣 고양이가 뜬다.  

### 4) filezilla 다운로드  

war 파일을 서버에 옮기려면 `filezilla`라는 프로그램이 필요하다.  
다운받고 실행한 뒤, 서버에 접속한다.  
war 파일을 드래그해서 옮기면 `permission denied`가 뜬다. 권한이 없기 때문이다.  
`root`계정으로 접속 후 `chmod -R 777 "권한 줄 폴더or파일"`으로 권한 주기  



리눅스는 각 파일에 대한 권한이 별도로 있어서 한번 살펴보는 것을 추천한다.  
뭣도 모르고 `chmod -R 777 /`로 모든 파일의 권한을 전체로 돌렸더니 `putty`와 `filezilla` 모두 접속이 안되는 상황이 벌어졌다.  
`putty`로 로그인하면 자꾸 `Network error: Connection refused`가 뜨는데, 이것 때문에 한참 해멨다. [여기](https://forums.aws.amazon.com/thread.jspa?threadID=113068)를 참고해서 전체 파일에 777 권한을 주는 것은 아주 위험하다는 것을 알 수 있었다. (절대 절대 시도하지마세요 인스턴스 다시 생성해야함)  
나는.. 인스턴스를 다시 생성해야만 했다.......  
그냥 원하는 폴더 or 파일에만 777 권한을 준다. 권한을 줄 때는 심사숙고하고 엔터를 치자.  



어쨌든 war 파일을 넣으면 톰캣이 실행된 상태면 war 파일이 풀어져서 폴더가 생성된다. 그러면 `ip주소:포트번호/프로젝트명(폴더이름)/` 으로 접근이 가능해진다.  
프로젝트 명을 없애는 방법은 [여기](https://song8420.tistory.com/169)를 참조했지만, 어차피 뒤로가면 이 코드가 필요 없어져서 삭제해야되니 패스한다.  

### 5) 포트번호 두개 나누기  

여기까지는 기본 프로젝트 배포 과정이였고, 이제 본격적인 작업으로 들어간다.  
[여기](https://huskdoll.tistory.com/575)에서 service 두개 만들라는 것을 그대로 따라했다. 대신 위에서 언급한대로 프로젝트명 없애는 코드는 삭제한다.  




`server.xml`에서 아래와 같이 작성한다.  
포트 번호를 각각 설정해주는 것과, `appBase`에 실제 프로젝트를 풀어놓을 폴더 경로를 작성하는 것이 중요하다.  
나머지는 뭐.. 그냥 기본적으로 적혀있을 것이다.  

```xml
<Service name="Catalina">
<Connector port="8888" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
<Engine name="Catalina" defaultHost="localhost">
<Realm className="org.apache.catalina.realm.LockOutRealm">
<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
	resourceName="UserDatabase"/>
</Realm>
<Host name="localhost" appBase="webapps/project1"
	unpackWARs="true" autoDeploy="true">
</Host>
</Engine>
</Service>

<Service name="Catalina2">
<Connector port="9999" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
<Engine name="Catalina" defaultHost="localhost">
<Realm className="org.apache.catalina.realm.LockOutRealm">
<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
	resourceName="UserDatabase"/>
</Realm>
<Host name="localhost" appBase="webapps/project2"
	unpackWARs="true" autoDeploy="true">
</Host>
</Engine>
</Service>
```

### 6) ROOT 폴더 생성  

위에 war 파일이 풀어질 때, 넣었던 파일명 그대로 폴더가 생성된다.  
따라서 `ROOT`로 이름을 바꿔준 뒤에 war 파일을 넣어주면 `ROOT` 폴더가 생성된다.  
`ROOT`로 파일이 풀어지면 프로젝트 명이 url 뒤에 붙지 않으므로 프로젝트 명 삭제하는 코드는 딱히 필요 없다.  
5번에서 `appBase` 경로를 포트 별로 나눠줬으므로 war 파일도 각각의 경로로 배포해준다.  
예시로 `project1`,`project2`를 들었지만 어떤 이름이어도 상관 없다.  
`webapps` 폴더 밑에 `project1`, `project2` 폴더를 각각 생성한 뒤, 각 폴더 안에 `ROOT.war`파일을 옮겨서 톰캣을 실행한다.  
이제 포트번호를 붙여 각각의 프로젝트에 접근할 수 있게 되었다.  

### 7) DB 연동하기(실패)  

홈페이지는 띄워지는데 DB 연동은 안해서 DB에 접근하는 페이지로 가면 오류가 뜬다.  
이제 DB를 연동해줘야한다.(쓰다보니 2편으로 넘어감)  



처음에는 Oracle을 EC2 인스턴스에 설치해서 구동하는 것을 생각했는데, 며칠을 시도했는데도 실패했고, ubuntu에서는 Oracle 설정이 까다롭다는 이야기가 많아서 RDS로 노선을 틀었다. 정확히 말하자면 Oracle 설치 후 서버에서 쿼리문을 적용하는 것까지는 성공했지만, 로컬에서 SQLDeveloper로 접속하는 거에서 계속 막혔다. (아시는 분 계시다면 메일로 좀...)  



RDS 서버로 DB를 구축하는 것과 인스턴스에 오라클을 설치하는 것의 차이가 궁금했는데, 찾아보니 DB설정을 하지 않고 편하게 사용하게 해주는 것이 RDS라고 이해했다. 다만 단점이라면 과금의 우려가 있다는 것..  
많이들 궁금해할 줄 알았는데 찾아보니 정보가 많이 없었다. [여기](https://dingrr.com/blog/post/rds를-써야-하나요-ec2에-설치하면-안되나요)를 참조했다.  
RDS도 프리티어가 있으니 프로젝트를 한다면 RDS를 사용하는 것을 추천한다. 굉장히 쉽고 빠르게 할 수 있었다.  
Window 서버라면 EC2에 오라클 + SQLDeveloper 설치가 간편한 것으로 알고 있다. 이 방법을 사용해도 괜찮을 것 같다.(실제로 조원은 그렇게 함)  



만약 EC2에 오라클 설치를 도전하고 싶다면 [여기](https://tttck88.tistory.com/47)를 참조하면 좋다. 많은 도움을 받았다.  
이 과정 중 헤맸던 것은 alien으로 rpm 파일을 deb으로 바꿀 때 굉~장히 오래 걸리니 인내심을 가지고 기다려야 한다는 것(필자는 ctrl+c로 계속 중단함- 뭐가 잘못된 줄 알고..),   
오라클 시스템 계정 비밀번호 치는 곳은 타이핑을 해도 빈칸으로 보여지니 당황하지말고 오타내지 말고 잘 쓰자.  
이 두개였다.  

### 8) RDS Oracle 서버 생성  

오라클 11g가 지원이 끊기면서 어떻게 하지 하고 검색을 많이 해봤는데 12를 써도 괜찮다는 이야기가 있어서 [여기](https://velog.io/@namgonkim/m1-mac-AWS-RDS-%EC%98%A4%EB%9D%BC%ED%81%B4-%EC%83%9D%EC%84%B1%ED%95%98%EA%B3%A0-SQL-Developer%EB%A1%9C-%EC%A0%91%EC%86%8D%ED%95%98%EA%B8%B0)를 참고했다.  
프리티어 체크하는 란을 아무리 찾아봐도 없길래 보니까 위에 배너에 파란색 기존 인터페이스로 전환합니다를 눌러야 뜬다. 모르고 돈 낼 뻔 했다(?)..  
언급했듯이 오라클 11g는 지원을 더이상 안해서 12 v26을 선택했다.  
사용자 이름과 비밀번호를 잘 적어두자.  

### 9) DB 옮기기  

AWS를 담당했던 조원에게 [여기](https://velog.io/@dsunni/%EB%A1%9C%EC%BB%AC-Oracle-DB%EB%A5%BC-AWS-RDS%EB%A1%9C-%EC%98%AE%EA%B8%B0%EA%B8%B0)처럼 DB export를 부탁했다. 다행히 친철한 조원님은 해주셨고 무사히 DB를 옮길 수 있었다.  
SQLDeveloper에서 각 프로젝트 별로 계정을 생성해서 그 계정으로 스크립트를 읽어 DB 전체를 복사했다. 권한은 모두 DBA를 주었다.  

### 10) 이미지 불러오기  

프로젝트 내부에 이미지를 저장하지 않았기 때문에, 현재 웹사이트는 모두 이미지가 엑박으로 뜬다.  외부 경로를 지정하여 war파일을 재배포 하면되는데, 리눅스는 윈도우와 파일 경로가 달라서 조금 애먹었다.  
[여기](https://websecurity.tistory.com/6)를 참조해 root로 부터 트리구조로 뻗어나간다는 것을 알 수 있었고, 필자는 `wepapps/project1`, `wepapps/project2` 폴더에 각각 이미지 폴더를 만들어주고 경로를 지정해주었다. 풀 경로는 `/usr/local/tomcat8.5/webapps/~` 이런 식으로 만들어진다.  

### 11) 그 외..  

#### ⓐ ubuntu 서버 내 톰캣 시작, 정지 명령어  

```
# 톰캣 시작
/usr/local/tomcat8.5/bin/startup.sh

# 톰캣 정지
/usr/local/tomcat8.5/bin/shutdown.sh
```

#### ⓑ ubuntu 서버 내 파일 소유자 변경 명령어  

권한을 매번 주기 귀찮으면 소유자를 바꿔놓으면 된다.  

```
chown -R 계정이름 "폴더이름"
```

#### ⓒ 에러 로그 보는 방법  

이클립스 톰캣만 사용하다보니 콘솔 로그 확인하는 게 이클립스 자체 콘솔이라고 생각해서 AWS 서버에서는 로그를 어떻게 확인하는지 헤맸다.  
알고보니 톰캣에 있는 로그를 보는 것이였다. 얼마나 서버에 대해 무지했는지 알 수 있었다..  
에러가 난 뒤에 명령어를 치면 짤려서 보이므로 미리 들어가놓고 에러가 나게끔(?)하면 전체 로그를 볼 수 있다.  

```
cd /usr/local/tomcat8.5/logs
tail -f catalina.out
```

#### ⓓ 엄청난 삽질의 경험  

파일 업로드 기능이 계속 오류가 나길래 처음에는 경로 문제인 줄 알았다.  
ajax로 드래그 앤 드랍으로 파일 업로드를 구현했는데 계속 ajax url이 404가 뜨면서 먹통이 되는 것이다.  
ajax url은 `/upload/uploadAjax` 였다.  
로컬에서는 문제없이 작동하고, 심지어 AWS EC2 window 서버에서도 잘 작동한 기능인데 ubuntu 배포를 하면서 안되길래 100% 경로문제구나! 라고 생각했다. 왜냐하면 바꾼 코드가 경로밖에 없기 때문이다.  
두번째는 권한 문제인가 싶었는데 `404`에러이다 보니 아니라고 결론지었다.  
너무 멘붕이었다. 감이 오는 것도 없었다.  
차근차근 아래와 같이 진행해보면서 생각해보니 404 에러는 컨트롤러를 찾지 못해서 발생한다는 것을 알게되었고, 경로 문제가 아니라 모종의 이유로 url에 맞는 컨트롤러를 찾고 있지 못하는 것이었다.  


1. `/upload/uploadAjax` url의 모든 data와 코드를 삭제하고 간소화시킨 뒤 컨트롤러까지 들어가는지만 확인 -> 안들어감  
2. `/upload/test` url과 맵핑되는 컨트롤러 생성 -> 안들어감
3. 근데 다른 url은 다 컨트롤러를 잘 찾아가는 상황..  

이 과정에서 검색을 하다가 [여기](https://www.kangtaeho.com/42)를 보고 머릿속에 전구가 켜졌다.  
`upload`라는 흔하디 흔한 어쩌면 존재할지도 모르는 폴더명..!  
혹시나해서 url을 `/uploadFile/uploadAjax`로 바꾸니 제대로 컨트롤러를 찾아갔다...  


## 3. 이어서..  

포스팅 하나로 작성하려고 했는데 뒤에 이어지는 내용이 조금 정리가 필요한 내용이여서 부득이하게 2편으로 나눠야 할 것 같다.  
2편은 각 프로젝트에 DB 연결을 하는 방법과 JNDI에 대해 정리할 것이다.  
또한 도움을 받은 참고 사이트들의 출처도 정리해야겠다.  
