---
title: 오픈소스, 줘도 못 먹나 후기
subtitle: 오픈소스로 팀의 개발 생산성 높이기
layout: post
tags: seminar
---

# Okky 오픈소스 세미나 오픈소스, 줘도 못 먹나 오픈소스로 팀의 개발 생산성 높이기

![oss_0]({{ "/assets/img/blogpost/oss_0.jpg"}})

Okky 창시자이신 허광남 님이 오픈소스 세미나를 하신다는 글을 Okky에서 보게됐다.

세미나 소개에 내가 사용하고 있거나, 궁금했던 오픈소스들이 많이 보였다.

이번기회에 내가 사용하고있는 오픈소스에 대해서 좀 더 알고 싶었고, 허광남님이 현업에서 어떤 오픈소스를 사용하고, 어떻게 도입하셨는지 궁금해서  신청했다!


## Intro

기록을 잘 남기고, 관리하는게 좋다.

테스트 자동화하면 코드의 품질이 높아진다.

어플리케이션 기능이 추가될 때마다 테스트케이스로 만들어놓으면 어디서 사이드 이펙트가 나는지 쉽게 알 수 있다.

__단위테스트__(TDD) (function기준)으로 만들어놓는게 좋다.

## 단계별 오픈 소스

![oss_1]({{ "/assets/img/blogpost/oss_1.png"}})

## 오픈소스 시작하기

__오픈소스 경험  = 시간 + 시행착오 + 성공실패__

그러니 일단 사용부터 (Getting started) 해라!

default setting 문서를 보고 시작하는게 좋다.

책/블로그 보고 시작할땐 버젼 맞추기

오픈소스 함부로 수정하면 안된다! => 오픈소스 커뮤니티를 추천

버전업 주기가 빠르기 때문

* 버전 팁

  **최신보다 1, 2 낮은 버전 사용**을 권장(서비스 시)
  
  LTS 권장
  
* LTS는 우분투에서 시작된 말

  => Long Term Support (2년동안은 버전을 올리지 않는다)
  
  => 최소 2년은 안심하고 써도 됨
  
* 최신 버전은 Blood Edge = > 유혈이 낭자하는(?) 최전선, 온갖 에러를 만날 수 있다.


## 팀에 오픈소스를 도입하고 싶을 때
```
1. 명분, 근거자료
2. 도입 전과 후 장점 어필(적은 노력으로 많은 효과 강조)
3. 영향이 적은 곳 부터 적용(실적이 잘 안나오는 부서부터 공략..)
4. 메인 비즈니스에는 경험이 쌓인 후 도입
```

## 오픈소스 기여방법
```
1. 일단 사용
2. 블로깅 (3년을 관련 오픈소스에 대한 포스팅을 하면 주변에서 전문가라 칭한다고 한다)
3. 설정 튜닝(configuration 튜닝)
4. 이슈 등록(contribution)
5. 문서 번역
6. fork&pull request
```

## 경험했거나 현재 사용하는 오픈소스 제품들
```
* 2000년 톰캣을 시작으로
* Ant
* Eclipse :에릭감마
* Junit :에릭감마&캔트 백이 만듬 => 부하테스트 할때 좋음 => Java Swing UI 로 만들어져있다.
* Jmeter
* Yona, Git, VSCode
* Jenkins, CentOS, VirtualBox
* Nginx, Node.js, Express.js
* MariaDB, ELK, Uptime
* Mocha, SonarQube, ZAP
```

**title 을 누르면 해당 오픈소스 홈페이지로 이동합니다**

### [Yona](https://github.com/yona-projects/yona)

프로젝트 관리도구

Yona는 Git을 서버로 관리해준다.

[네이버 오픈소스](https://naver.github.io/) 였었다(지금은 나옴)

이슈트래커 + Gix(SVN)

무료 (Git private는 돈을 내야하지만 이건 무료)

Email 설정을 하면 보다 깊게 사용할 수 있다. 

Email 설정시 imap.folder = "Inbox"로 설정되어있는지 확인할 것(꼭 "Inbox"여야한다. 소문자면 안됨)



### [GIT](https://git-scm.com/)

분산 버전 관리 시스템

리눅스 토발즈가 만들었고 홈페이지도 운영한다.

유용하게 사용한 상황별 커맨드를 정리 

https://okdevtv.com/mib/git


### [VSCode](https://code.visualstudio.com/)

electron js 기반으로 만들어진 오픈소스 편집기가 브래킷, 아톰, vscode

VSCode 안의 git 기능을 강추(특히 누가 선택한 라인을 작성했는지 (git blame 기능) 볼수 있다)

GitLens Plugin과 Quokka Plugin(유료)가 좋다.

Quokka Plugin은 변수값 수정시 다른 곳 해당 변수명의 변수 값도 같이 수정해주는 기능이있다.

https://okdevtv.com/mib/vsc

https://github.com/Microsoft/vscode-tips-and-tricks <<어마어마한 vscode 팁과 트릭이있다. 

### [Jenkins](https://jenkins.io/)

허드슨 만든 개발자가 Oracle에서 나와서 Jenkins를 만들었다.

지속적인 통합(Conituous Integration) 

=> Git과 Jeckins를 잘 사용하면 완료 한달전 파트 합칠때의 유혈 사태(?)를 방지할 수 있다.

빌드 & 모니터 자동화

Java로 만들어짐

에러 로그를 다 확인할 수 있어 좋다. 

https://okdevtv.com/mib/jenkins

### [Virtualbox](https://www.virtualbox.org/)

테스트용도 (많은 이미지가 있지만 Mac OS는 지원하지 않는다)

https://okdevtv.com/mib/virtualbox

### [Nginx](https://www.nginx.com/)

아파치보다 가벼움, 설정도 가벼움

로드밸런싱 지원

https://okdevtv.com/mib/nginx

### [MariaDB](https://mariadb.com/)

MySQL 만든 개발자가 오라클의 삽질로 오라클에서 나와서 만든 DB

둘째 딸 이름이 Maria 라 MariaDB

MySQL과 굉장히 흡사하다(MySQL과 동일한 API) 

심지어 프로세스 명이 mysql.exe이다 (MySQL은 mysqld.exe...)

https://okdevtv.com/mib/mariadb


### [ELK Stack](https://www.elastic.co/kr/elk-stack) 

elestic회사에서 만든 오픈소스 Elesticsearch(검색엔진) java 로 되어있다.

ElasticSreach + Logstash + Kibana + Beats

=> 엑세스 로그를 모니터링(차트로 볼 수 있게 되어있다)

https://okdevtv.com/mib/elk/elk6

### [Uptime](https://github.com/fzaninotto/uptime)

=> 1분마다 서버를 체크해준다.

메일설정하면 서버상태 보내준다 (서버가 죽었다거나..)

굉장히 편리

### [Mocha](https://mochajs.org/)

Junit 대신 자바 유닛테스트 하기위함

### [SonarQube](https://www.sonarqube.org/)

소스 정적 분석 도구

코드의 품질을 높일 수 있다

중복 코드 퍼센테이지와 취약한 코드를 볼 수 있다.

프로젝트 root에 sonar-project.prop 두면 결과가 reporting 된다.

https://okdevtv.com/mib/sonar


### [ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)

취약점 분석 자동 툴

반드시 본인 사이트에만 할 것 (다른 사이트에 하면 큰일남)

XSS, SQL Inejction등 OWASP 취약점으로 있는건 다 테스트 한다.

https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project

https://okdevtv.com/mib/zap

#### 추가

노드 + 크롬 디버깅은 NiM inspect manager 플러그인이 좋다!

## 후기 

<<작성중>>














