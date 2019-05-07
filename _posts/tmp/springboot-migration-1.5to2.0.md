---
layout: post
title: springboot 1.5 에서 2 로 버전 업그레이드
---
[공식 가이드](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)는 이쪽이지만
이렇게 했을때 왠지 잘 안됐었기 때문에 작성하게 되었습니다.
필수 사항 : JDK 8 이상
---
우선, 버전을 마이그레이션 할 때 
몇가지 라이브러리들은 버전을 업그레이드 해주지 않으면 에러가 납니다
그리고 한방에 버전을 쭉 올려버리면 오류가 납니다(왜 그런지 아시는 분은 코멘트 부탁드리겠습니다..)
그래서 `1.5 -> 2.0 -> 2.1` 이렇게 올렸습니다.

## 순서
### 1. 그래들 버전이 4.4 가 아닐경우 4.4-all 로 변경
gradle > wrapper > gradle-wrapper.properties의
```
distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip
```
로 수정합니다.

### 2. `build.gradle`에 ependency-management를 추가
```gradle
apply plugin: 'io.spring.dependency-management'
runtime("org.springframework.boot:spring-boot-properties-migrator")
```
### 3. springboot 버전 바꾸기 
```gradle
springBootVersion = '2.0.5.RELEASE'
```
아래부턴 쓰는 라이브러리가 다양하기 때문에 
버전 업그레이드로 인한 어떤 라이브러리 설정의 변화로 발생하는 오류를 고치는 방법을 작성하진 않았습니다.
대신 어떻게 고쳐야 하는지에 대한 문서는
* [common-application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
* [공식문서 링크](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#servlet-specific-server-properties)
이곳을 참고하시면 될 것 같습니다.

### 4. 빌드
만약 컴파일 오류가 난다면 해당되는 오류를 고칩니다.(필자는 log4j 오류가 나서 log4j2로 고쳤음)

### 5. `application.properties` 수정
빌드 깨진 것 확인 후 해당 설정을 `application.properties`에서 적절하게 고칩니다.
(저는 datasource 쪽에서 자꾸 오류가 났습니다)

### 6. bean overriding
만약 
`org.springframework.beans.factory.support.Bean Definition Override Exception` 오류가 발생하면
 `application.properties` 에 아래 코드를 추가합니다
```gradle
spring.main.allow-bean-definition-overriding=true
```
이것은 bean overriding을 허용한다는 것 입니다.

만약 `The server time zone value ‘KST’ is unrecognized or represents more than one time zone : mysql-connector-java` 오류가 발생하면
DB url의 DB명 뒤에 `?characterEncoding=UTF-8&serverTimezone=UTC` 를 추가합니다
```
jdbc:mysql://아이피:포트/DB명?characterEncoding=UTF-8&serverTimezone=UTC
```
### 7. 2.0 -> 2.1
빌드 및 run이 성공적으로 되면 `build.gradle`에서  `2.1.3.RELEASE` 로 springboot 버전을 변경합니다.
```gradle
springBootVersion = '2.1.3.RELEASE'
```
그 후 위의 작업을 반복하며 오류를 잡으면 됩니다.
2.1 버전 부턴 mysql 드라이버 명이 바꼈습니다.
```gradle
기존 : com.mysql.jdbc.Driver
바뀐것 :  com.mysql.cj.jdbc.Driver
```
### 8. properties-migrator 제거
빌드 및 run이 성공적으로 되면 build.gradle에서 추가해뒀던 아래 코드를 제거 합니다.
```gradle
runtime("org.springframework.boot:spring-boot-properties-migrator")
```

내용이 잘못되었거나 보충해야 될 부분이 있다면 코멘트 부탁드리겠습니다🙏

추가
(19.5월 기준) 만약 swagger를 쓰는데 마이그레이션 시 오류 날 경우 2.9.1로 올려주면 됩니다 :)

참고자료
* [spring-boot-migration-java](https://altkomsoftware.pl/en/blog/spring-boot-migration-java/)
* [공식문서](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#servlet-specific-server-properties)