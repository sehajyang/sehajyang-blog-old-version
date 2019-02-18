---
layout: post
title: mybatis association
tags: [tip, mybatis, spring] 
category : tip
comments: true
---
# mybatis association

Lombok 사용
Service 생략

### File
* Shop.java
* ShopDetail.java
* ShopMapper.xml

Shop.java
~~~java
@Data
public class Shop {
    private String shopno;
    private String id;
    private String pwd;

    private ShopDetail shopDetail; //변수명 앞문자 capital 주면 안됨(getter, setter 때문)
}
~~~

ShopDetail.java
~~~java
public class ShopDetail {
    private String shopno;
    private String name;
    private String price;
    private String count;
}
~~~

ShopMapper.xml
~~~xml
<!-- 중략 -->
<mapper namespace="com.test.mapper.ShopMapper">

    <resultMap id="ShopMap" type="com.test.domain.Shop">
        <result property="shopno" column="shopno" />
		<result property="id" column="id" />
		<result property="pwd" column="pwd" />
        <association property="shopDetail" javaType="com.test.domain.ShopDetail">
            <result property="shopno" column="shopno" />
		    <result property="name" column="name" />
		    <result property="price" column="price" />
		    <result property="count" column="count" />
        </association>
    </resultMap>

    <select id = "selectTestQuery" resultMap = "ShopMap">
        select s.*, sd.* from shop s join shopdetail sd on s.shopno = sd.shopno
    </select>
~~~

### Result
~~~console
Shop[(shopno=1, id=aa, pwd=1234, shopDetail=shopDetail(shopno=1, name=김뭐뭐, price=1000, count=100))]
~~~

