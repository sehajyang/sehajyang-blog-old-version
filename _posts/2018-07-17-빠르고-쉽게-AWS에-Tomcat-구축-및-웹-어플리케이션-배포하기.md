
## 목차

1. RDS 생성 및 세팅
2. sqldeveloper에 RDS 계정 연결 및 테이블 구성
3. EC2 생성 및 세팅 
4. war 파일 배포



* 사용하는 프로그램

  Sql Developer, Putty, Eclipse, FileZila



시작하기 전 AWS 계정이 있고, 사용하는 프로그램이 미리 설치 되어 있다고 가정합니다.




## EC2 생성 및 세팅

### EC2 생성



![aws1](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/1.JPG)

서비스 항목에 들어가면 위와 같은 화면을 볼 수 있는데

우리가 이 중 사용할건 컴퓨팅의 EC2 와 RDS 입니다
EC2에는 Tomcat8을 설치할 것이고, RDS에는 oracle를 구성할 것 입니다.

![aws2](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/2.JPG)



EC2를 클릭 후 인스턴스 시작을 누르게 되면 위와같은 화면이 나옵니다

ubuntu를 선택후 프리티어 사용가능 표시가 있는 인스턴스 유형을 선택한 후 검토및 시작을 누릅니다.

7단계로 건너뛰게 됩니다.

![aws3](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/3.JPG)

새 키 페어 생성을 선택후 이름을 정합니다.
그 뒤 키페어 다운로드를 하게 됩니다 ~.pem 이며 분실시 재발급이 되지 않으므로 잘 보관하도록 합시다.

1~2분정도 기다리면 EC2 인스턴스를 사용할 수 있게됩니다.



### EC2 셋팅

만들어진 EC2 인스턴스에 java나 tomcat등을 설치해줘야합니다.

puttygen을 실행합니다.

Load를 선택후 All file로 탐색해 아까 생성한 새로운 키 페어.pem 파일을 선택합니다.  

![aws4](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/4.JPG)

그 후 Save private key를 선택하고 key.ppk를 저장할 위치 및 이름을 지정합니다.

putty 를 실행합니다. 

HostName에 ubuntu@퍼블릭DNS 를 넣어줍시다.

![aws4](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/5.JPG)

퍼블릭DNS 는 인스턴스에서 볼 수 있습니다

Port 는 SSH, 22입니다. 

왼쪽사이드바 중 Connection > SSH > Auth 를 들어갑니다. 

![aws6](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/6.JPG)


Private key file for ...의 Browse 에서 아까 생성한 ppk 파일을 넣어줍니다. 

초기화면으로 돌아와서 Seved Sessions을 해 접속하기 편하게 저장해두고 Open을 선택합니다.

![aws7](https://github.com/yunjey0/yunjey0.github.io/blob/master/assets/img/awspost/7.JPG)

위와 같은 화면이 뜬다면 성공입니다.
