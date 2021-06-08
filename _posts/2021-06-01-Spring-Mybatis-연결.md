---
title: Spring Mybatis 연결
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Java
toc: true
toc_sticky: true
toc_label: 목차
description: Spring Mybatis 연결
article_tag1: Spring Mybatis 연결
article_tag2:
article_tag3:
article_section: Spring Mybatis 연결
meta_keywords: Java, Spring
last_modified_at: 2021-06-07T00:00:00+08:00
---

## jUnit 설정

### JDK와 jUnit 설정

```xml
<modelVersion>4.0.0</modelVersion>
<groupId>com.myspringtest</groupId>
<artifactId>aid</artifactId>
<name>MySpringTest</name>
<packaging>war</packaging>
<version>1.0.0</version>
<properties>
	<java-version>1.8</java-version>
	<org.springframework-version>4.3.30.RELEASE</org.springframework-version>
	<org.aspectj-version>1.9.6</org.aspectj-version>
	<org.slf4j-version>1.7.30</org.slf4j-version>
</properties>
```

### JUnit dependency버전을 4.12로 변경

```xml
<!-- Test -->
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.12</version>
	<scope>test</scope>
</dependency>
```

@Test
테스트해야하는 메소드 위에 추가하는 애노테이션
jUnit은 해당 메소드를 테스트 코드로 간주하고 테스트를 진행함

@Before
테스트 코드 실행에 앞서 준비되어야하는 내용을 추가하는 애노테이션
@Test 메소드 실행 전에 실행됨

@After
테스트 코드 실행 완료 후에 실행되는 메소드에 추가하는 애노테이션

org.junit.Assert.assertxxx
테스트 중에 발생하는 값을 확신하는 용도로 사용
테스트 도중에 특정 값이나 상태를 예상하고 체크하는 용도로 사용

### JDBC 연결 테스트

Mysql Connecter 등록

```xml
<!-- Mysql Connector -->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.21</version>
</dependency>
```

### 테스트 (MySQLConnectionTest.java)

패키지 생성 후 파일 생성 및 작성 > 생성된 파일에서 오른쪽 마우스 > Run As > JUnit Test 클릭

DriverManager 등이 에러가 날 경우 프로젝트의 Java version 확인 필요 (Java Build Path 등)

MySQLConnectionTest.java

```java
package com.myspringtest.study;

import java.sql.Connection;
import java.sql.DriverManager;

import org.junit.Test;

public class MySQLConnectionTest {

	// mysql 생성시 인코딩 문제
	// https://proinlab.com/archives/1147
	// 현재 db 캐릭터셋 확인 : SELECT * FROM INFORMATION_SCHEMA.SCHEMATA;
	// serverTimezone=Asia/Seoul
	private static final String DRIVER = "com.mysql.cj.jdbc.Driver";
	private static final String URL = "jdbc:mysql://localhost:3306/study?useUnicode=yes&characterEncoding=UTF-8&serverTimezone=UTC";
	//jdbc:mysql:주소:포트/DB명
	private static final String USER = "User명";
	private static final String PW = "패스워드";

	@Test
	public void testConnection() throws Exception{
		Class.forName(DRIVER);
        // AutoCloseable 인터페이스를 구현한 타입의 변수
        try(Connection conn1 = DriverManager.getConnection(URL, USER, PW);
            Connection conn2 = DriverManager.getConnection(URL, USER, PW);) {

            System.out.println("===== mysql connection test start =====");
            System.out.println(conn1);
            System.out.println(conn2);
            System.out.println("===== mysql connection test end =====");

        } catch(Exception e) {
            e.printStackTrace();
        }
	}
}
```

실행방법

![결과](https://user-images.githubusercontent.com/83876951/120511453-65fea580-c405-11eb-941a-5770a2b0b8f2.png)

결과

```console
===== mysql connection test start =====
com.mysql.cj.jdbc.ConnectionImpl@6193932a
com.mysql.cj.jdbc.ConnectionImpl@647fd8ce
===== mysql connection test end =====
```

에러 발생시,
프로젝트 우클릭 > Properties > Java Build Path > Libraries > Add Library > JUnit 추가

![image](https://user-images.githubusercontent.com/83876951/121114354-86d45a00-c84e-11eb-9c6f-4e9e02a03d1a.png)

# Mybatis 연동

## Step 1 : pom 추가

- MySQL : MySQL 라이브러리
- MyBitis : MyBitis 프레임워크
- MyBitis-Spring : Spring과 MyBitis를 연결하는 라이브러리
- Spring-jdbc : jdbc 라이브러리
- Spring-test : 스프링과 MyBitis가 정상적으로 연동되었는지 확인 라이브러리

```xml
<!-- MyBatis -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.5.3</version>
</dependency>

<!-- MyBatis-Spring -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>2.0.6</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>5.2.12.RELEASE</version>
</dependency>

<!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
<dependency>
	<groupId>commons-dbcp</groupId>
	<artifactId>commons-dbcp</artifactId>
	<version>1.4</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-beans</artifactId>
	<version>5.2.12.RELEASE</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>5.2.12.RELEASE</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>5.2.12.RELEASE</version>
</dependency>
```

### root-context.xml 수정

- /src/main/webapp/WEB-INF/spring/**root-context.xml** 파일의 namespace 에서 **aop, beans, context, jdbc, mybatis-spring** 를 선택

![네임스페이스확인](https://user-images.githubusercontent.com/83876951/121049208-77272800-c7f2-11eb-8428-a5e87cb923f6.png)

- Source 탭으로 이동하여 다음을 추가.

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
  <property name="url" value="jdbc:mysql://localhost:3306/study?characterEncoding=utf-8&amp;serverTimezone=UTC" />
  <property name="username" value="root" />
  <property name="password" value="root" />
</bean>

<!-- Mysql <-> Mybatis를 연결해주는 객체 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis-config.xml" />
<!-- <property name="mapperLocations" value="classpath:/mapper/**/*Mapper.xml" /> -->
</bean>

<bean class="org.mybatis.spring.SqlSessionTemplate" id="sqlSession" destroy-method="clearCache">
	<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

DB url, username, password 같은 정보는 보안상 따로 파일을 만들어서 변수를 사용하여 입력하는게 좋은 방법.
<context:property-placeholder location="/WEB-INF/database.properties"/> 태그로 database.properties 파일을 해당 xml파일로 import를 하여 dataSource 빈에 property에 변수를 값으로 설정.

sqlSessionFactory 빈에는 mybatis 설정파일과 sql문을 관리하는 mapper xml파일의 경로와 데이터베이스 연결정보들을 가지고 있는 dataSource 객체를 property 값으로 가지고 있음.

mybatis-config.xml, member-config.xml 파일은 **src/main/resources** 디렉토리 밑에 생성하여 만든 설정파일로써 해당경로를 클래스패스로 인식하기 때문에 값을 넣어줄때 classpath:를 써줘야 인식 가능.

sqlSessionFactory는 데이터베이스 연결과 sql문 실행에 대한 모든 것을 가진 가장 중요한 객체.

### mybatis-config.xml 파일 생성

![image](https://user-images.githubusercontent.com/83876951/121053098-f9fdb200-c7f5-11eb-9968-e98f74626993.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

### DataBase와 MyBatis 연동테스트 (root-context.xml에 입력한 코드 확인)

다음 코드가 에러가 날 경우
[링크]({{ "https://blog.naver.com/PostView.nhn?blogId=pyj721aa&logNo=221261302683&parentCategoryNo=&categoryNo=49&viewDate=&isShowPopularPosts=true&from=search" }}){:target="\_blank"} 참고.

![image](https://user-images.githubusercontent.com/83876951/121055015-dcc9e300-c7f7-11eb-9bb6-81e7c013aa18.png)

DataSourceTest.java 파일 생성

```java
package com.myspringtest.study;

import java.sql.Connection;

import javax.inject.Inject;
import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

// SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해 주면
// JUnit이 테스트를 진행하는 중 테스트가 사용할 어플리케이션 컨텍스트를 만들고 관리하는 작업을 해준다.
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"file:src/main/webapp/WEB-INF/spring/**/root-context.xml"})
public class DataSourceTest {

	@Inject
	private DataSource ds;

	@Test
	public void testConnection() throws Exception{
		try(Connection conn = ds.getConnection()){
			System.out.println("===== connection test start =====");
			System.out.println(conn);
			System.out.println("===== connection test end =====");
		}catch(Exception e){ e.printStackTrace(); }
	}
}
```

MyBatisTest.java 파일 생성

```java
package com.myspring.starter;

import javax.inject.Inject;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/*.xml" })
public class MyBatisTest {

	@Inject
	private SqlSessionFactory sqlFactory;

	@Test
	public void testFactory() {
		System.out.println(sqlFactory);
	}

	@Test
	public void testSession() throws Exception {
		try (SqlSession session = sqlFactory.openSession()) {
            System.out.println("===== connection test start =====");
			System.out.println(session);
            System.out.println("===== connection test end =====");
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
