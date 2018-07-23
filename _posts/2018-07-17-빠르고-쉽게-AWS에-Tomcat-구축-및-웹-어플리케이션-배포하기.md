
## 목차

1. RDS 생성 및 세팅
2. sqldeveloper에 RDS 계정 연결 및 테이블 구성
3. EC2 생성 및 세팅 
4. java 및 tomcat 설치
5. war 파일 배포

<hr>

* 사용하는 프로그램

  Sql Developer, Putty, Eclipse, FileZila



시작하기 전 AWS 계정이 있고, 사용하는 프로그램이 미리 설치 되어 있다고 가정합니다.




## RDS 생성 및 세팅

![aws13]({{ "/assets/img/awspost/13.JPG"}})

aws 서비스중 데이터베이스 > RDS 를 선택한 후  create database를 합니다.

![aws12]({{ "/assets/img/awspost/12.JPG"}})

oracle 선택 후 하단의 프리티어 옵션에 체크하고 다음단계로 넘어갑니다.

![aws14]({{ "/assets/img/awspost/14.JPG"}})

DB 인스턴스 이름과 유저 이름 및 암호를 설정합니다. 

다음 단계에서 특별히 설정할 것은 없습니다(퍼블릭 액세스는 가능으로 선택). 

Create database를 합니다.

DB 인스턴스가 생성이 되고, 사용할수 있는 상태가 되기까진 10분 이상 걸립니다. 

![aws15]({{ "/assets/img/awspost/15.JPG"}})

사용 가능한 상태가 되었습니다.



## sqldeveloper에 RDS 계정 연결 및 테이블 구성

이제 SQL Developer에 연결해보도록 하겠습니다.

생성한 DB 인스턴스의 세부정보를 들어가 엔드포인트 정보를 복사합니다.

![aws16]({{ "/assets/img/awspost/16.JPG"}})

SQL Developer 데이터베이스 접속 새로만들기를 합니다.

![aws17]({{ "/assets/img/awspost/17.JPG"}})

호스트 이름은 아까 복사해둔 엔드포인트 입니다. 

사용자 이름 및 비밀번호는 RDS DB 인스턴스 생성시 했던것과 동일하게 하면 됩니다.

![aws18]({{ "/assets/img/awspost/18.JPG"}})

접속 성공

기존 local에 table 만들고 데이터 입출력 하던것 처럼 이젠, 아마존 RDS DB에 연결해 사용하면 됩니다.



## EC2 생성 및 세팅

### EC2 생성

![aws1]({{ "/assets/img/awspost/1.JPG"}})

서비스 항목에 들어가면 위와 같은 화면을 볼 수 있는데

우리가 이 중 사용할건 컴퓨팅의 EC2 와 RDS 입니다
EC2에는 Tomcat8을 설치할 것이고, RDS에는 oracle를 구성할 것 입니다.

![aws2]({{ "/assets/img/awspost/2.jpg"}})



EC2를 클릭 후 인스턴스 시작을 누르게 되면 위와같은 화면이 나옵니다

ubuntu를 선택후 프리티어 사용가능 표시가 있는 인스턴스 유형을 선택한 후 검토및 시작을 누릅니다.

7단계로 건너뛰게 됩니다.

![aws3]({{ "/assets/img/awspost/3.JPG"}})

새 키 페어 생성을 선택후 이름을 정합니다.
그 뒤 키페어 다운로드를 하게 됩니다 ~.pem 이며 분실시 재발급이 되지 않으므로 잘 보관하도록 합시다.

1~2분정도 기다리면 EC2 인스턴스를 사용할 수 있게됩니다.



### EC2 셋팅

만들어진 EC2 인스턴스에 java나 tomcat등을 설치해줘야합니다.

puttygen을 실행합니다.

Load를 선택후 All file로 탐색해 아까 생성한 새로운 키 페어.pem 파일을 선택합니다.  

![aws4]({{ "/assets/img/awspost/4.jpg"}})

그 후 Save private key를 선택하고 key.ppk를 저장할 위치 및 이름을 지정합니다.

putty 를 실행합니다. 

HostName에 ubuntu@퍼블릭DNS 를 넣어줍시다.

![aws4]({{ "/assets/img/awspost/5.JPG"}})

퍼블릭DNS 는 인스턴스에서 볼 수 있습니다

Port 는 SSH, 22입니다. 

왼쪽사이드바 중 Connection > SSH > Auth 를 들어갑니다. 

![aws6]({{ "/assets/img/awspost/6.JPG"}})


Private key file for ...의 Browse 에서 아까 생성한 ppk 파일을 넣어줍니다. 

초기화면으로 돌아와서 Seved Sessions을 해 접속하기 편하게 저장해두고 Open을 선택합니다.

![aws7]({{ "/assets/img/awspost/7.JPG"}})

위와 같은 화면이 뜬다면 성공입니다.



## Java 및 Tomcat 설치

#### Java 설치

계속해서 java와 tomcat을 설치해보도록 하겠습니다.

일단 root 권한으로 접속하겠습니다

```
sudo passwd root
```

패스워드 설정을 합니다

```
su
```

설정한 패스워드를 입력하고 root 권한으로 실행을 합니다. 

만약을 대비해 root 권한으로 실행하기보다 sudo를 사용하는 방법이 있는데, 

저는 root권한으로 진행하도록 하겠습니다.



```
apt-get update
```

업데이트를 시켜줍니다.

EC2 ubuntu 엔 java가 설치되어있지 않기 때문에 java를 설치하겠습니다.

```
apt-get install openjdk-8-jre-headless
```

설치 후 

```
java -version
```

으로 설치가 잘 되었는지 확인해봅니다.



#### Tomcat 설치 및 EC2 포트추가 

```
apt-get install tomcat8
```

톰캣을 설치합니다.

기본 설치 path는 /var/lib/tomcat8 입니다.

![aws8]({{ "/assets/img/awspost/8.JPG"}})

8080 포트를 열어주기 위해 네트워크 및 보안> 보안그룹으로 갑니다.

보안그룹 생성을 합니다. 이름 및 설명은 자유입니다.

![aws9]({{ "/assets/img/awspost/9.JPG"}})

하는김에 80번과 1521포트도 추가합니다.

![aws11]({{ "/assets/img/awspost/11.JPG"}})

보안그룹 변경을 눌러 방금 추가한 보안그룹을 체크 후 보안그룹 할당을 눌러 추가시켜 줍니다.

그 후, 인스턴스를 재시작 후 인스턴스 IPv4 퍼블릭 IP로 접속합니다.

```
http://IPv4퍼블릭IP:8080
```
![aws10]({{ "/assets/img/awspost/10.JPG"}})

위와같은 화면이 보인다면 성공입니다.



## war 파일 배포

FileZila를 실행합니다. 

왼쪽 상단 아이콘을 선택합니다. (이미지의 빨간 동그라미 친 아이콘)

![aws19]({{ "/assets/img/awspost/19.JPG"}})

노란 형광표시된 항목을 채웁니다.

호스트 IP는 EC2 인스턴스의 퍼블릭 IP 입니다.

사용자는 ubuntu고, 키파일은 아까 생성한 ppk 파일을 잡아주면 됩니다.

그 후 연결을 선택합니다.

![aws20]({{ "/assets/img/awspost/20.JPG"}})

연결이 되면 home/ubuntu 가 보이게 됩니다. 

이 안에 웹프로젝트 war 파일을 끌어다 놓으면 전송이 됩니다. 

​						 ![aws21]({{ "/assets/img/awspost/21.JPG"}})

putty로 EC2 인스턴스의 /home/ubuntu 밑에 war 파일이 이동된 걸 볼 수 있습니다. 

```
mv 이동하려는파일.war /var/lib/tomcat8/webapps
```

위의 명령어로 이동하려는 war 파일을 tomcat8 밑의 webapps에 넣어주면 알아서 war 파일이 배포가 됩니다.

주소창에서 잘 배포가 되었는지 확인합니다.

```
http://EC2 인스턴스의 퍼블릭 IP:8080/웹 프로젝트 context Name
```

만약 webapps 아래 war 파일이 자동으로 풀리지 않았다면

```
service tomcat8 restart
```

 톰캣을 restart 시켜줍니다. (10여분 가량 소요)

