---
title: Spring Mybatis DB 연결 1
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
description: Spring Mybatis DB 연결 1
article_tag1: Spring Mybatis DB 연결 1
article_tag2:
article_tag3:
article_section: Spring Mybatis DB 연결 1
meta_keywords: Java, Spring
last_modified_at: 2021-06-03T00:00:00+08:00
---

**Tip:** 지난 자료 참고에서 제일 하단 **POM 최종정리**를 확인  
그외 **root-context.xml**, **mybatis-config.xml** 설정도 확인 가능  
[Spring Mybatis 연결 jUnit 테스트]({{ "" | relative_url }}{{% post_url 2021-06-04-Spring-Mybatis-연결-JUnit-테스트.md}}){:target="\_blank"}  
[Spring Mybatis DB 연결 1]({{ "" | relative_url }}{{% post_url 2021-06-03-Spring-Mybatis-DB연결-#1.md}}){:target="\_blank"}
{: .notice--info}

## 1. 최종 샘플 파일

[최종샘플파일(SpringTest_20210628.zip)]({{ site.baseurl }}/assets/files/2021/SpringTest_20210628.zip){:target="\_blank"}
{: .notice--info}

## 2. 현재 페이지 최종 파일 생성 참고

![image](https://user-images.githubusercontent.com/83876951/123401029-57a94100-d5e1-11eb-85d2-4ba0f14b75f5.png)

## 3. 시작

### `root-context.xml` 수정

namespace에 **`context, mybatis-spring`** 추가

![image](https://user-images.githubusercontent.com/83876951/123773418-9c95e600-d907-11eb-8234-04af1a1e3fbd.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<!-- Root Context: defines shared resources visible to all other web components -->

	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	  <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy" />
      <property name="url" value="jdbc:log4jdbc:mysql://192.168.10.100/kotraDB?characterEncoding=utf-8&amp;serverTimezone=UTC" />
      <property name="username" value="kotra"/>
      <property name="password" value="Kotra!@"/>
	</bean>

	<!-- Mysql <-> Mybatis를 연결해주는 객체 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
	    <property name="configLocation" value="classpath:mybatis-config.xml" />
		<property name="mapperLocations" value="classpath:/mapper/**/*Mapper.xml" />
	</bean>

 	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
	</bean>

	<mybatis-spring:scan base-package="com.**.dao.*"/>

	<!-- <context:component-scan base-package="com.member.dao"></context:component-scan> -->
	<context:component-scan base-package="com.member.service"></context:component-scan>

</beans>
```
