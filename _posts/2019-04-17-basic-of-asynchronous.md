---
layout: post
title: 비동기 기초 정리 
comments: true
categories : [Etc]
tags: [asynchronous]
---

매주 목요일마다 진행하는 개발관련 사내스터디에서 비동기에 대한 발표를 들었는데 내용이 좋아서 정리를 했다.  
원본(발표자) 슬라이드는 [이곳](https://www.slideshare.net/MinChulLee6/asynchronous-101-1-128879123?ref=https://www.slideshare.net/MinChulLee6/slideshelf)에 있고
2부는 [링크](https://www.slideshare.net/MinChulLee6/asynchronous-101-2) 에서 볼 수 있다 😄

**목차**
* TOC
{:toc}

## 1. IT Revolution 과 동시접속자 증가
* 직접회로를 쓰는 IC >  PC > Internet > Mobile로 발전해나갔다.
* 그 과정에서 사용자의 폭발적인 증가
* 사용자의 증가 => 동시 접속자 증가 이를 해결하려면?
    * 연산을 더 빠르게
        * 하드웨어 Upgrade(램을 더 달거나, 코어를 늘리거나..), H/W 적인 해결
        * Low Level Language (어셈블러 C 등) => but 어렵고 힘듦, S/W 적인 해결
    * 동시성을 늘린다
        * Parallelism (병렬 처리) => CPU 더 달아서 듀얼코어.., H/W 적인 해결
        * Concurrency (동시성)

## 2. Thread and Process
* Thread 
    * Process보다 적은 overhead와 자원 사용 => Process안에서 Thread 영역만(그 안엔 Stack 영역만이 존재) 생성하면 되기 때문
    * 멀티코어 활용이 어려움 => 구현하려면 굳이 멀티코어로 해줘야 됨(알아서 안해줌)
    * Data 공유가 쉬움
    * 공유 자원(양날의 검)
* Process
    *  Thread보다 더 많은 자원 사용(cpu, memory) 사용, 생성시 Code, Data, Stack, Heap영역을 다 만듦
    * 멀티코어를 활용할 수 있음 => OS가 멀티코어를 알아서 해줌
    * Data 동기화가 어려움

## 3. Deadlock 
*생략*
* 추천도서 : [7가지 동시성 모델](http://www.hanbit.co.kr/store/books/look.php?p_code=B3745244799)

## 4. Sub Routine 과 Coroutine
* 서브루틴 (Sub Routine)
    * 루틴이 다 끝날 때 까지 아무것도 못함 (리턴할 때 까지 대기해야됨)
    * 함수, 메소드
* 코루틴 (Coroutine)
    * 루틴 진행 중간에 멈춰서 특정 위치로 돌아갔다가 다시 원래 위치로 돌아와 나머지 루틴을 실행
    * 비동기와 결합하면 더 강력해진다

## 5. 동기(Synchronous)와 비동기(Asynchronous) 
* 동기
    * sync를 맞추다
    * 모든 루틴을 순차적으로 진행
    * **시간관계**를 동시에 또는 동일하게 유지하는 처리방식 (but 시간관계 는 어떠한 관점으로 보느냐에 따라 다름)  
* 비동기
    * sync를 맞추지 않아도 되는 처리 방식 (맞출수도 아닐수도 있다)
    * 몇몇 루틴을 비순차적으로 진행할 수도 있음
    * 즉, 무조건 sync 다 맞추지 않아야 비동기! 이건 아님
* 다양한 곳에서 동기와 비동기를 활용
    * Serial 통신에서 활용
        * 동기 : 클럭에 맞춰 시작과 끝
        * 비동기: 시작과 끝을 정함 (ex)101로 시작하고 001로 끝내라) 
    * CPU 연산에서 활용(pipelining)

## 6. Non-block
* 프로세스를 비동기식으로 만들기 위해서 등장
* Block
    * 함수가 끝날 때 까지 기다림
    * 함수의 결과를 return 함
* Non Block
    * 함수를 바로 return 함
    * 함수의 결과를 따로 저장함
        * Main이 Non-block Object에 주기적으로 결과리턴을 물어봄(물어보는 건 blocking function)
        * 리턴이 False면 기다리지않고 다른 작업을 하고 또 물어봄
        * 리턴이 True가 되면 Non-block Object는 result를 보냄
        * 이 과정에서 언제 끝나는지 계속 물어보는 게 필요함 => Event Loop

## 7. Event Loop
* 외부 환경에서 이벤트를 받음
* 이벤트 발생하면 해당 이벤트에 대한 루틴을 실행시킴

## 8. Non block + Event Loop → Async
* Non block 함수가 완료되었는지 아닌지 검사하는 event loop
    * 작업이 완료된지 아닌지를 주기적으로 물어봄
    * 더 효율적인 Async Loop는 어떤 작업의 상태가 바뀌었는지 select 한다
    * 더더 효율적인 Async Loop는 작업의 상태가 바뀌면 event loop에게 알려준다(커널이 지원해줘야됨) *Observer Pattern?*
    * 이런것들을 I/O multiplexing 기술들이라 한다.

## 9. Async
* Async loop에 non block이 아닌 block 함수가 들어오면?
    * Block 함수가 실행되는 동안 event loop가 멈춘다 => 성능저하
        * Block 함수를 Thread나 Process로 concurrency하게 실행해서 non block처럼 만든다
        * But 공유자원 접근의 위험 발생 => deadlock 블라블라..
        * 코루틴!

## 10. Coroutine + Async
* 코루틴은 루틴 실행하고 중단했다가 다시 시작할 수 있다
* 동시성(concurrent) 구현  
    * Single Thread에서 실행되기 때문에 shared data 접근 충돌 걱정이 사라짐
* 동시성이 있지만 또 다른 thread나 process 만들 필요가 없음
    * 메모리 save
    * Thread, Process를 만드는 것에 대한 오버헤드가 없음
* But Blocking 함수 또는 CPU Bound 작업을 할 때 성능이 저하됨
    * 한 작업을 할때 다른 작업을 처리할 수 없기 때문
    * 따라서 Blocking 함수는 thread로 처리
    * 그 Blocking 함수가 공유자원을 참조해야 하는 경우에는 동시성에 대한 지식 및 S/W 적 안전장치 마련해야됨 

## 11. CPU Bound, IO Bound
* CPU Bound : 연산이나 처리량이 많은 경우
    * 문자열 연산, 사칙연산, 영상/이미지 처리
    * 동시성을 늘리려면 돈을 더 붓던가(h/w 업그레이드), 저급 언어로 코딩
* I/O bound : 입력과 출력이 많은 경우

## 12. 언제 비동기를 써야할까
* 처리순서의 시간관계가 관련이 없는 작업이 많을 때
* **I/O 작업이 많을때**
* 여러개 작업을 처리해야 할 때
