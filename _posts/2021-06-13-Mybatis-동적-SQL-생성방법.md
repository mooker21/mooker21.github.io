---
title: Mybatis 동적 SQL 생성 방법
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Java
  - Spring
toc: true
toc_sticky: true
toc_label: 목차
description: Mybatis 동적 SQL 생성 방법
article_tag1: Spring
article_tag2: Mybatis
article_tag3:
article_section: Mybatis 동적 SQL 생성 방법
meta_keywords: Java, Spring, Mybatis
last_modified_at: 2021-06-13T00:00:00+08:00
---

[참고]({{ "https://atoz-develop.tistory.com/entry/MyBatis-%EB%8F%99%EC%A0%81-SQL-choose%EC%99%80-set%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%99%EC%A0%81-SQL-%EB%A7%8C%EB%93%A4%EA%B8%B0?category=840168" }}){:target="\_blank"} 사이트를 통해 복사 정리.

## MyBatis 동적 SQL 종류

### if test

```xml
<if test="조건">SQL</if>
```

### choose

```xml
<choose>
    <when test="조건1">SQL</when>
    <when test="조건2">SQL</when>
    <otherwise>SQL</otherwise>
<choose>
```

### where

`<where>` 안의 조건식에서 일치하는 조건이 있으면 WHERE절을 만들어 반환하고 없으면 만들지 않는다.

```xml
<where>
    <if test="조건1">SQL</if>
    <if test="조건2">SQL</if>
</where>
```

### trim

조건에 따라 SQL이 반환되면 SQL문의 앞부분에서 `prefixOverrides`에 지정된 문자열과 일치하는 문자열을 제거하고 `prefix`로 지정한 접두어를 붙인다.

```xml
<trim prefix="접두어" prefixOverrides="문자열|문자열">
    <if test="조건1">SQL</if>
    <if test="조건2">SQL</if>
</trim>
```

### set

UPDATE문의 SET절을 만들때 사용.  
`<set>` 안의 조건식에서 일치하는 조건이 있으면 SET절을 만들어 반환하고 없으면 만들지 않음.  
SET절의 항목이 여러 개일 경우 자동으로 콤마(,)를 붙임.

```xml
<set>
    <if test="조건1">SQL</if>
    <if test="조건2">SQL</if>
</set>
```

#### MyBatis 동적 SQL - `<set>` 사용 예

```xml
<update id="update" parameterType="map">
    update PROJECTS
    <set>
        <if test="title != null">PNAME=#{title},</if>
        <if test="content != null">CONTENT=#{content},</if>
        <if test="startDate != null">STA_DATE=#{startDate},</if>
        <if test="endDate != null">END_DATE=#{endDate},</if>
        <if test="state != null">STATE=#{state},</if>
        <if test="tags != null">TAGS=#{tags},</if>
    </set>
    where PNO = #{no}
</update>
```

startDate, endDate만 전달되면 다음과 같이 동적 SQL문이 생성

```sql
update PROJECTS
SET STA_DATE = #{startDate},
     END_DATE = #{endDate}
where PNO = #{no}
```

### foreach

IN (값, 값) 을 만들때 편리

```xml
<foreach
    item="항목"
    index="인덱스"
    collection="목록"
    open="시작문자열"
    close="종료문자열"
    separator="구분자">
</foreach>
```

- item : 한 개의 항목을 가리키는 변수 이름 지정
- index : 인덱스 값을 꺼낼때 사용할 변수 이름 지정
- collection : java.util.List 구현체나 배열 객체 지정
- open : 최종 반환값의 접두어 지정
- close : 최종 반환값의 접미어 지정
- separator : 구분자 문자열 지정

#### foreach 사용예

```xml
<select id="" ...>
	SELECT *
	FROM table
	where id IN
	<foreach collection="list" item="map" separator="," open="(" close=")">
		#{map}
	</foreach>
</select>
```

```xml
String[] array;
<select id="" ...>
	SELECT *
	FROM table
	where id IN
	<foreach collection="array" item="map" index="i" separator="," open="(" close=")">
		#{map[i]}
	</foreach>
</select>
```

```xml
ArrayList<String> list = new ArrayList<>();
<select id="" ...>
	SELECT *
	FROM table
	<where>
		<foreach collection="list" item="map" separator="or" open="(" close=")">
			id = #{map}
		</foreach>
	</where>
</select>
```

### bind

변수 생성시 사용

```xml
<bind name="변수명" value="값"/>
```
