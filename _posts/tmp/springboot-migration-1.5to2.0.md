---
layout: post
title: springboot 1.5 에서 2 로 버전 업그레이드
---
[공식 가이드](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)는 이쪽이지만
이렇게 했을때 왠지 잘 안됐었기 때문에.. 작성하게됨
필수 사항 : JDK 8 이상
---
우선, 버전을 마이그레이션 할 때 
몇가지 라이브러리들은 버전을 업그레이드 해주지 않으면 에러가 남
그리고 한방에 버전을 쭉 올려버리면 오류가 난다..
1. 1.5 -> 2.0
2. 2.0 -> 2.1
이렇게 했다(왜인지 모르겠지만 1.5 -> 2.1로 올리면 오류남..)

순서 : 
1. 그래들 버전이 4.4 가 아닐경우 4.4-all 로 변경
gradle > wrapper > gradle-wrapper.properties의
```
distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip
```
로 수정

2. `build.gradle`에 아래를 추가
```gradle
apply plugin: 'io.spring.dependency-management'
runtime("org.springframework.boot:spring-boot-properties-migrator")
```
2.1 springboot 버전을 바꿈 1.5 -> 2.0.5.RELEASE
3. 빌드
4. 빌드 깨진 것 확인 후 해당 설정을 `application.properties`에서 적절하게 고침 (필자는 datasource 쪽에서 자꾸 오류가 남)
5. 만약 컴파일 오류가 난다면 해당되는 것 고침 (필자는 log4j 오류가 나서 log4j2로 고쳤음)

`application.properties` 에서 이것도 추가해줌 
```gradle
spring.main.allow-bean-definition-overriding=true
```

해당 문서는 [여기](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
이렇게 properties 안의 내용을 바꾸면 됨 
여튼 old property를 바꾸란 말임 
[공식문서 링크](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#servlet-specific-server-properties)

만약 KST어쩌고 하면서 datasource 오류나고 그러면 
DB url의 DB명 뒤에 `?characterEncoding=UTF-8&serverTimezone=UTC` 를 추가해주면 됨

빌드 다 잘 되고 run 되면 build.gradle에서 
```gradle
runtime("org.springframework.boot:spring-boot-properties-migrator")
```
이것 지우...기 전에 2.1.3.RELEASE 로 springboot 버전을 변경하고
다시 빌드하고 오류잡고 그럼 됨..
2.1 부턴 mysql 드라이버 명도 바꼈음 
```gradle
com.mysql.jdbc.Driver 에서 com.mysql.cj.jdbc.Driver 로..
```
끝

추가
(19.5월 기준) swagger쓰는데 마이그레이션 시 오류 날 경우 2.9.1로 올려주면 오류안남
참고자료
* [spring-boot-migration-java](https://altkomsoftware.pl/en/blog/spring-boot-migration-java/)