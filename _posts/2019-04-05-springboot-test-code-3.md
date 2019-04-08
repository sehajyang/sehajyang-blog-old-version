---
layout: post
title: Springboot 에서 test code 작성하기 3편 - assetThat이 중복되는 테스트와 Spock
tags: [Test]
categories : [Java]
comments: true
---

>#### Springboot 에서 test code 작성하기 시리즈
* [Springboot 에서 test code 작성하기 1편 - 통합테스트(MVC)](https://sehajyang.github.io/java/2019/04/05/springboot-test-code-1.html)
* Springboot 에서 test code 작성하기 2편 - 단위테스트(Service) 추후 작성
* [Springboot 에서 test code 작성하기 3편 - assetThat이 중복되는 테스트와 Spock](https://sehajyang.github.io/java/2019/04/05/springboot-test-code-3.html)

### 1. 그 외의 메소드 테스트와 Spock
프로덕션엔 CommonUtil이라는 클래스가 있다.   
그 클래스엔 문자열을 조작한다던가, 리스트를 조작하거나 어떠한 값을 검증하는 등의 메소스들이 있는데 그 클래스 내의 메소드들이 잘 동작하는지 테스트하기 위해선 많은 assertThat 이 필요했고, 그러한 중복코드를 없애기 위해 검색을 하던 중 [Spock 소개 및 튜토리얼](https://jojoldu.tistory.com/228)라는 튜토리얼을 보게 되었다.  

>**BDD(Behaviour-Driven Development)**는 TDD가 진화한 테스트 코드 작성 방식이며  
사람이 쉽게 이해하기 쉬운 작성, 즉 테스트의 동작 및 의도를 직관적으로 알 수 있다는 장점이 있다.  
**Spock는 groovy 로 작성된 프레임워크**인데 JVM에서 실행되며 **테스팅을 하기 위한 BDD 프레임워크**이다.   
따라서 Given-When-Then-Expect_Where 로 시나리오를 작성하게 된다.  
또한 __테스트 실패 시 상세하게 이유가 출력 된다__  

예를들어 **특정 시간의 예약 가능 여부를 True/False 로 리턴**하는 `checkReserveYN` 라는 함수가 있다고 가정한다.  
일반적으론 그 함수를 테스트 할 때
~~~java
@Test
public void Test() {
    boolean data1 = CommonUtil.checkReserveYN("월요일 09:00~19:00, 토요일 13:00~16:00");
    assertTrue(data1);

    boolean data2 = CommonUtil.checkReserveYN("금요일 09:00~24:00");
    assertTrue(data2);

    boolean data3 = CommonUtil.checkReserveYN("월,화 13:00~16:00");
    assertTrue(data3);
}
~~~
이렇게 작성하게 되는데 테스트 케이스 하나를 만들 때 마다 중복이 발생하고 있는 것을 알 수 있다.  
하지만 spock를 사용하면 이 부분을 해결할 수 있다.

~~~groovy
    @Unroll
    def "#time 시간의 예약 가능 여부 : #result"() {
        expect:
        result as Boolean == CommonUtil.checkReserveYN(time as String)

        where:
        time                                   | result
        "월요일 09:00~19:00,토요일 13:00~16:00" | true
        "금요일 09:00~24:00"                   | false
        "월,화 13:00~16:00"                    | true
    }
~~~
`@Unroll` 어노테이션은 메소드 명에 where의 객체값을 대입해 테스트를 수행해 위의 테스트의 결과는  
![image1]({{ "/assets/img/blogpost/springboot-test-code-3-1.JPG"}})  
이렇게 나타난다.  

만약 테스트가 실패한 경우 아래와 같이 상세하게 테스트 실패 이유를 출력한다.  
![image2]({{ "/assets/img/blogpost/springboot-test-code-3-2.JPG"}})  

또한 request 나 session값 등은 메소드 필드에 그냥 선언해서 사용할 수 없다.  
즉 이렇게 사용하면 안된다.  
~~~groovy
def "세션 Id가 없으면 false를 리턴해야한다"() {
        given:
        MockHttpSession session = null
        MockHttpSession sessionNossId = null
        session.setAttribute("ssId", "1")

        expect:
        result as boolean == CommonUtil.SessionIdCheck(ss as MockHttpSession)

        where:
        ss | result
        session | true
        sessionNossId | false
}

~~~
위 코드는 `groovy.lang.MissingPropertyException : No such property session` 가 발생한다.  
생성한 session 객체를 공유 하기 위해선 `@shared` 어노테이션을 사용하거나 static 변수로 선언하는 방법이 있다.  
`@shared`는 메소드간의 객체를 공유할 수 있게 해준다.   
하지만 공유 객체를 만들고 사용하는 것 보다 더 좋은 방법이 있다.   

~~~groovy
    def "세션 Id가 없으면 false를 리턴해야한다"() {
        expect:
        result as boolean == CommonUtil.SessionIdCheck(ss as MockHttpSession)

        where:
        ss << {
            MockHttpSession session = new MockHttpSession()
            MockHttpSession sessionNossId = new MockHttpSession()
            session.setAttribute("sessionId", "1")
            [session, sessionNossId]
        }()
        result << [true, false]
    }
~~~
where 내 에서 session 객체를 생성한다.   
위와 같이 작성하면 `@shared` 없이 깔끔하게 코드를 작성할 수 있다.  
또한 `result << [_,false]` 등으로 무시할 수 있다.  
자세한 내용은 [링크](https://www.pluralsight.com/guides/introduction-to-testing-with-bdd-and-the-spock-framework) 에서 확인할 수 있다.  


참고자료
* [Spock Framework Reference Documentation](http://spockframework.org/spock/docs/1.0/spock_primer.html)
* [Spock 소개 및 튜토리얼](https://jojoldu.tistory.com/228) 
* [Spock으로 테스트코드를 짜보자](http://woowabros.github.io/study/2018/03/01/spock-test.html) 