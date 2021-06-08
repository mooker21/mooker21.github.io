---
title: Spring Mybatis DB 연결
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
description: Spring Mybatis DB 연결
article_tag1: Spring Mybatis DB 연결
article_tag2:
article_tag3:
article_section: Spring Mybatis DB 연결
meta_keywords: Java, Spring
last_modified_at: 2021-06-09T00:00:00+08:00
---

### 기본설정

**Tip:** 지난 자료 참고에서 제일 하단 **POM 최종정리**를 확인  
그외 **root-context.xml**, **mybatis-config.xml** 설정도 확인 가능  
[Spring Mybatis 연결 jUnit 테스트]({{ "2021-06-01-Spring-Mybatis-연결-JUnit-테스트.md" }}){:target="\_blank"}
{: .notice--info}

pom.xml 에 MyBatis Sql 콘솔로그 출력을 위한 log4jdbc-log4j2 라이브러리 추가

```xml
<!-- Mybatis log -->
<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4.1 -->
<dependency>
	<groupId>org.bgee.log4jdbc-log4j2</groupId>
	<artifactId>log4jdbc-log4j2-jdbc4</artifactId>
	<version>1.16</version>
</dependency>
```

### root-context.xml 수정

1. DataSource 부분은 Sql 로그 확인을 위해 log4jdbc 설정 추가

2. Mybatis SqlSessionFactoryBean 에서는 Mybatis 설정 기능을 활용할 수 있도록 설정 파일을 읽어오는 부분과 SQL 문을 작성해둘 mapper.xml 파일 설정.

3. SqlSessionTemplate는 기본적인 트랜잭션 관리나 쓰레드 처리 안정성 보장, DB의 연결과 종료를 관리.  
   SqlSessionTemplate을 등록해두면 개발자가 직접 트랜잭션 관리나 DB 연결, 종료를 해야 하는 작업을 직접 처리 함.

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy" />
  <property name="url" value="jdbc:log4jdbc:mysql://localhost:3306/study?characterEncoding=utf-8&amp;serverTimezone=UTC" />
  <property name="username" value="root" />
  <property name="password" value="root" />
</bean>

<!-- Mysql <-> Mybatis를 연결해주는 객체 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis-config.xml" />
	<property name="mapperLocations" value="classpath:/mappers/**/*Mapper.xml" />
</bean>

<bean class="org.mybatis.spring.SqlSessionTemplate" id="sqlSession" destroy-method="clearCache">
	<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

### 테이블 생성

```sql
-- 테이블
CREATE TABLE `TB_TEST`(
      `USER_ID` VARCHAR(10) NOT NULL,
      `USER_PW` VARCHAR(10),
      `USER_NM` VARCHAR(10),
    PRIMARY KEY (`USER_ID`)
);
-- 데이터
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NM) VALUES ( 'id1', 'pw1', 'name1');
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NM) VALUES ( 'id2', 'pw2', 'name2');
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NM) VALUES ( 'id3', 'pw3', 'name3');
```
