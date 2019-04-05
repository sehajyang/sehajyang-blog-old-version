---
layout: post
title: Springboot 에서 test code 작성하기 3편 - 그 외(Spock)
---

>#### Springboot 에서 test code 작성하기 시리즈
* [Springboot 에서 test code 작성하기 1편 - 통합테스트(MVC)]()
* Springboot 에서 test code 작성하기 2편 - 단위테스트(Service) 추후 작성
* [Springboot 에서 test code 작성하기 3편 - 그 외(Spock)]()

### 3. 그 외의 메소드 테스트
프로덕션엔 CommonUtil이라는 클래스가 있다.
그 클래스 내의 메소드들이 잘 동작하는지 테스트하기 위해선 많은 assertThat 이 필요했고, 그러한 중복코드를 없애기 위해 검색을 하던 중  [Spock 소개 및 튜토리얼](https://jojoldu.tistory.com/228)라는 튜토리얼을 보게 되었다.

참고자료
* [jojoldu 님의 Spy 사례1 - 테스트 대상 Mocking 하기](https://jojoldu.tistory.com/239) 
* [Spock으로 테스트코드를 짜보자](http://woowabros.github.io/study/2018/03/01/spock-test.html) 