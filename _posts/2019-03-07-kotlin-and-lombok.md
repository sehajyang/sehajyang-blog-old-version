---
layout: post
title: Kotlin 에서 Lombok을 사용할 수 없는 문제
categories : [Etc]
comments: true
tags: [tips]
---
최근 자바 스프링부트로 되어있는 사내 프로젝트에 Kotlin을 일부분 도입하려다 대차게 실패했다.  
사내 프로젝트에 Lombok을 사용하고 있었는데, Kotlin이 Lombok을 사용한 Bean Class에 접근하지 못하는 것 이었다.

당시 
~~~
Kotlin: Cannot access '가가' it is private member of '나나'
~~~
대략 이런 에러 메세지를 띄우며 컴파일이 되지 않았다.

처음엔 Java로 짜여진 class의 private object에 접근을 못해서 나는 에러인 줄 알았으나 Lombok 문제였다.  
[kotlin-doesnt-see-java-lombok-accessors](https://stackoverflow.com/questions/35517325/kotlin-doesnt-see-java-lombok-accessors/35530223#35530223)  
위 링크에서 이유를 알 수 있었다.

결론적으로 
> JVM에서 Kotlin이 먼저 컴파일되고 JAVA가 컴파일되기 때문에  
Kotlin이 Java의 Lombok으로 된 클래스에 접근하려 하니 getter setter가 만들어지지 않아서 접근을 못하는 것 이었다.
Lombok은 컴파일시 동작 하는 어노테이션이기 때문이다.   
강제로 컴파일러가 Java 먼저 컴파일하게 바꿀 순 있지만 그렇게되면 Java에서 Kotlin코드를 사용할 수 없다.

Kotlin과 Java가 100호환은 맞지만 Java로 만들어진 third-party들과 호환 되는건 아닌 것 이다.
Lombok을 사용한 Java 파일들을 전부 Kotlin으로 바꾸지 않는 한 Kotlin을 현재 진행 중인 프로젝트에 적용 하긴 힘들 것 같다.

#### 한줄요약
* Kotlin과 Lombok을 같이 쓸 수 없다.
* 정말 쓰고싶으면 Lombok을 사용한 Java 파일을 Kotlin 파일로 바꿔야 된다.
