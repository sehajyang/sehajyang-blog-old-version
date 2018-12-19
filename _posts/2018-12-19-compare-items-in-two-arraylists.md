---
layout: post
title: 두개의 리스트의 아이템을 비교해 추가 삭제
tags: [java]
---

최근 두개의 ArrayList의 아이템을 비교해, 추가되는 아이템과 삭제되는 아이템을 구분할 필요가 있어서 만들었다.

<가정>
list1 = [a, b, c, d] =>새로 받아온 ArrayList
list2 = [a, b, e] => 기존 DB 있던 Data를 담은 ArrayList

```
ArrayList<String> resultList = compageAndDel(list1, list2);
for (int i = 0; i < resultList.size(); i++) {
    System.out.println("Inst " + resultList.get(i)); // c,d 출력 => DB에 없던, 새롭게 추가될 데이터
}

ArrayList<String> resultList2 = compageAndDel(list2, list1);
for (int i = 0; i < resultList2.size(); i++) {
    System.out.println("Del " + resultList2.get(i)); // e 출력 => 기존 DB에 있었지만 삭제된 데이터
}

private static ArrayList<String> compageAndDel(ArrayList<String> target, ArrayList<String> source) {
    ArrayList<String> tmpArr = new ArrayList<>();
    tmpArr.addAll(target);
    for (String item : source) {
        if (target.contains(item) == true) {
            //일치하는 아이템을 지움
            tmpArr.remove(item);
        }
    }
    return tmpArr;
}

```
위의 코드로 새로 추가될 아이템과 DB에 있었지만 삭제될 아이템을 구분할 수 있다.
