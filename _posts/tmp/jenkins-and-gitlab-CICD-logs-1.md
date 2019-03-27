---
layout: post
title: Gitlab 과 Jenkins 연동한 배포 자동화
---

부제 : DevOps 뉴비의 젠킨스 도입기
* TOC
{:toc}

### 구축 환경
OS : ubuntu  
WAS : tomcat8  
JAVA : JDK 8  
Git Web Platform : GitLab  
AWS : EC2 t2.micro 인스턴스 사용  
배포 프로젝트 : springboot  

### 추가적으로 받은 젠킨스 플러그인
* GitLab Plugin
* Gradle Plugin
* Publish Over SSH
* Slack Notification Plugin

### 목표
gitlab master 브랜치에 push 혹은 `merge request accept` 가 발생하면   
jenkins가 빌드 후 서버에 배포  
빌드 시작 및 결과는 slack 으로 알림  

### 발생한 문제

#### GitLab 인증문제
사내 프로젝트는 private repository로 되어있다.
private repository 일 경우 외부에서 접근하기 위해 인증이 필요하다.  
인증 방법으론 대략 3가지가 있다. 
1. rsa 키페어를 만들어서 사용
2. gitlab의 access token을 받아서 사용
3. gitlab 계정 인증을 사용

##### 1. rsa 키페어를 만들어서 사용
[링크](https://taetaetae.github.io/2018/02/08/github-with-jenkins/)를 참고해서 했다. 
문제없이 잘 생성해서 잘 인증받을 수 있었다.

##### 2. gitlab의 access token을 받아서 사용

##### 3. gitlab 계정 인증을 사용


#### Webhook 문제
그 뒤 gitlab webhook으로 푸쉬 요청을 받아야 되는데 404 에러가 났다.

> internal error  

구글링 해 보니 보통 네트워크 방화벽 문제 라고 한다.  
jenkins server는 회사 IP만 접근 할 수 있었기 때문에 gitlab에서 push 노티를 보내도 못받는 것 이다. 그래서 8080 포트를 열고 secret token을 추가했다.    
jenkins server 가 auth 오류를 response 했다.  

webhook url은 이런 모양이다.
~~~
http://젠킨스ID:젠킨스PWD@젠킨스호스트/project/프로젝트명
~~~
하지만 URL에 젠킨스 ID, PWD를 싣는다는게 영 꺼름칙하므로  
jenkins 관리 >  system configure > gitlab Enable authentication for '/project' end-point 에 체크를 해제하고 secret 토큰을 주면 된다.  
secret 토큰은 jenkis 생성한 프로젝트 아이템의 설정의 
(이미지)
여기서 generate 할 수 있다.    
generate 한 secret token을 webhook sercret token에 넣어주면 된다.  
그 후 webhook test 에서 push 이벤트를 발생시키니 성공적으로 빌드가 되는 걸 확인했다. 

슬랙은 과거에 jenkins에 붙여본 적이 있어서 어렵지 않게 할 수 있었다.  
(슬랙 붙인거 어쩌고저쩌고 이미지 및 설명)  


라이브 서버에 바로 배포하는것은 무서우니 배포할 테스트 서버를 새로 만들었다.  
배포를 위한 java와 tomcat을 설치했는데 아차 java를 아무생각없이 10을 깔아버린 것이었다.  
부랴부랴 java10을 지우고 java8을 깔았다.  
추후에 이게 문제가 되어 테스트 서버를 새로 만들고 java와 tomcat을 다시 설치했다.  

이제 ssh로 파일을 옮길 차례였다.   
처음엔 jenkins 서버에서 테스트 서버로 접속을 시도해봤다.  

인증문제인줄 알았는데 인증문제면 permission denide 라도 떠야되는데 테스트 서버에선 아무 메세지도 보내고있지 않았다.  
처음엔 명령어를 잘못 쓴 줄 알았다.  
그래서 ping도 날려봤다 하지만 무응답이었다.  
하지만 서버는 죽어있지 않고 잘 돌아가고 있었다.  
명령어도 바꿔보고 했는데, 알고보니 private ip로 접속해야 되는 것이었다.  
아무튼 private ip로 바꾸니 테스트 서버가 permission denide 메세지를 주었다.(드디어 응답을 했다)  
~~~
ssh -i 키.key 테스트서버ip
~~~
그래도 키 인증 오류가 났다  
AWS 자습서에 가보니 키.key 가 아니고 키.pem를 넣으라고 한다  
(이미지)
잘 되었다  

그러던 차에 Publish Over SSH 라는게 있다는 걸 알게되었다.  
스크립트로 ssh mv 파일 이런걸 하려던 차에 한번 써볼까 한게 문제의 시작이었다.  
일단 jenkins 설정에서 ssh 관련 셋팅을 잘 했다. test도 success 가 떴다.  
build를 돌려보니 연결이 잘 된다고 뜬다.  
기쁜마음으로 보낼 war 파일이 있는 path와 받을 테스트서버의 path를 잘 적었다.  
하지만 빌드를 아무리 돌려봐도 SSH: Transferred 0 file(s) 가 떴다.  
젠킨스 서버의 war파일은 사라지고 테스트 서버는 받은게 없다.  
정말 이상한 일이었다.   
마치 돈을 준사람은 있고 받은사람은 없는 것 같은 상황이었다.  
구글링 해보면 절대경로, 상대경로 문제라고 한다.   
일단 source files path 문제라고 생각했다.   
그래서 **/* 로 옵션을 줘 봤다.  
/var/lib/jenkis/ 아래의 모든 파일이 transfer 됐다.  
source files를 여지껏 못찾고 있었던 것이었다.

젠킨스 서버의 환경변수를 까봤다. ${JENKIS_HOME} 이 없었다. 아 환경변수가 없구나   
jenkins 의 홈은 보통 /var/lib/jenkins이다.  

난 환경변수가 안잡혀있으니 그냥 절대경로를 줬다. 그게 문제였다.  
jenkins 의 setting 메뉴 최상위에 보면 jenkins home path가 잡혀있고,  
ssh transfers의 source file 에 적는 path는 /var/lib/jenkins/workspace/프로젝트명/ 아래에 들어가게 되는 것 이었다..  
그것도 모르고 source files에 절대경로랍시고 /var/lib 이하생략을 줬던것이었다.  
remote directory path는 / 로 줬다.  
기본 path 는 /home/유저명/ 인 것 같다.  
빌드를 돌려보자 성공적으로 파일을 테스트 서버의 /home/유저명 아래로 보낸것을 알 수 있었다.  
빌드 후 조치에 받은 파일을 mv로 옮기고 tomcat restart 명령을 실행했다. (모든 명령은 sudo로 진행되었다)  
remove prefix 옵션을 주지 않으면 war 파일을 찾으러 가는 경로에있는 모든 디렉토리를 같이 보낸다.  


빌드가 잘 되는걸 볼 수 있다.  

그 외에도 읽기 쓰기 권한 문제라던가 그러한 사소한 문제가 있었지만 chmod로 적절히 조치를 취했다.  

*작성중*








