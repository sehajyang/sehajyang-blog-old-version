---
layout: post
title: gitlab에 이미 push된 commit 삭제
categories : [Tip]
comments: true
tags: [git]
---
gitlab, github등에 취소해야 되는 commit이 있는데 이미 push 해버렸을 때 
git bash에서
~~~
git reset --hard 되돌아가고자 하는 커밋주소
~~~
* 커밋 주소는 git log 로 알 수 있다
* reset 옵션으론 hard, soft, mixed 가 있다 [링크](https://www.devpools.kr/2017/02/05/%EC%B4%88%EB%B3%B4%EC%9A%A9-git-%EB%90%98%EB%8F%8C%EB%A6%AC%EA%B8%B0-reset-revert/)

그 후
~~~
git push origin +브랜치명
~~~
브랜치 명 앞에 `+`를 붙여야 한다(덮어씌운다는 의미)
`+` 없이 push 해버리면 최근 커밋을 pull 하라며 에러가 난다. 

만약 
~~~
 ! [remote rejected] master -> master (pre-receive hook declined)
~~~
이런 에러가 났다면 해당 
>project settings > Repository > Protected Branches 
에서 unprotected로 해당 브랜치를 설정해주면 된다 (보통 master에 걸려있음)

