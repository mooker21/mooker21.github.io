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
{: .notice--info}

## 1. 최종 샘플 파일

[최종샘플파일(SpringTest_20210628.zip)]({{ site.baseurl }}/assets/files/2021/SpringTest_20210628.zip){:target="\_blank"}
{: .notice--info}

## 2. 현재 페이지 최종 파일 생성 참고

![image](https://user-images.githubusercontent.com/83876951/123401029-57a94100-d5e1-11eb-85d2-4ba0f14b75f5.png)

## 3. 시작

### 기본설정 (Log설정)

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

### `web.xml` 수정

filter 추가

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>

	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

</web-app>
```

### `servlet-context.xml` 수정

<context:component-scan base-package="com.member.controller" /> 추가

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans:beans xmlns="http://www.springframework.org/schema/mvc"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:beans="http://www.springframework.org/schema/beans"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd
http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->

    <!-- Enables the Spring MVC @Controller programming model -->
    <annotation-driven />

    <!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
    <resources mapping="/resources/**" location="/resources/" />

    <!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
    <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    	<beans:property name="prefix" value="/WEB-INF/views/" />
    	<beans:property name="suffix" value=".jsp" />
    </beans:bean>

    <context:component-scan base-package="com.member.controller" />

</beans:beans>
```

### `root-context.xml` 수정

1. DataSource 부분은 Sql 로그 확인을 위해 log4jdbc 설정 추가

2. Mybatis SqlSessionFactoryBean 에서는 Mybatis 설정 기능을 활용할 수 있도록 설정 파일을 읽어오는 부분과 SQL 문을 작성해둘 mapper.xml 파일 설정.

3. SqlSessionTemplate는 기본적인 트랜잭션 관리나 쓰레드 처리 안정성 보장, DB의 연결과 종료를 관리.  
   SqlSessionTemplate을 등록해두면 개발자가 직접 트랜잭션 관리나 DB 연결, 종료를 해야 하는 작업을 직접 처리 함.

![image](https://user-images.githubusercontent.com/83876951/121302417-9ded7780-c934-11eb-9687-ddf40b59a267.png)

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy" />
  <property name="url" value="jdbc:log4jdbc:mysql://localhost:3306/study?characterEncoding=utf-8&amp;serverTimezone=UTC" />
  <property name="username" value="아이디" />
  <property name="password" value="패스워드" />
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

<context:component-scan base-package="com.member.dao"></context:component-scan>
<context:component-scan base-package="com.member.service"></context:component-scan>

```

### 테이블 생성

```sql
-- 테이블
CREATE TABLE `TB_TEST`(
      `USER_ID` VARCHAR(10) NOT NULL,
      `USER_PW` VARCHAR(10),
      `USER_NAME` VARCHAR(10),
    PRIMARY KEY (`USER_ID`)
);
-- 데이터
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NAME) VALUES ( 'id1', 'pw1', 'name1');
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NAME) VALUES ( 'id2', 'pw2', 'name2');
INSERT INTO TB_TEST(USER_ID, USER_PW, USER_NAME) VALUES ( 'id3', 'pw3', 'name3');
```

### Log 관련 파일 생성

**src/main/resources** 아래에 다음 파일 생성

1.  **log4jdbc.log4j2.properties**
2.  **logback.xml**

log4jdbc.log4j2.properties

```
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <!-- log4jdbc-log4j2 -->
    <logger name="jdbc.sqlonly" level="DEBUG" />
    <logger name="jdbc.sqltiming" level="INFO" />
    <logger name="jdbc.audit" level="WARN" />
    <logger name="jdbc.resultset" level="ERROR" />
    <logger name="jdbc.resultsettable" level="ERROR" />
    <logger name="jdbc.connection" level="INFO" />

</configuration>
```

### log4j Level 수정

수정 후 Eclipse Console에서 쿼리 관련정보 확인 가능

![image](https://user-images.githubusercontent.com/83876951/121312990-e90d8780-c940-11eb-8f5c-457ff747ea3c.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration PUBLIC "-//APACHE//DTD LOG4J 1.2//EN" "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

	<!-- Appenders -->
	<appender name="console" class="org.apache.log4j.ConsoleAppender">
		<param name="Target" value="System.out" />
		<layout class="org.apache.log4j.PatternLayout">
			<param name="ConversionPattern" value="%-5p: %c - %m%n" />
		</layout>
	</appender>

	<!-- Application Loggers -->
	<logger name="com.starter.spring">
		<level value="info" />
	</logger>

	<!-- 3rdparty Loggers -->
	<logger name="org.springframework.core">
		<level value="info" />
	</logger>

	<logger name="org.springframework.beans">
		<level value="info" />
	</logger>

	<logger name="org.springframework.context">
		<level value="info" />
	</logger>

	<logger name="org.springframework.web">
		<level value="info" />
	</logger>

	<!-- Root Logger -->
	<root>
		<priority value="info" />
		<appender-ref ref="console" />
	</root>

</log4j:configuration>
```

### MyBatis 설정 파일 ( mybatis-config.xml ) 생성

**src/main/resources** 아래 **mybatis-config.xml** 파일 생성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<typeAliases>
		<typeAlias type="com.member.dto.MemberVO" alias="memberVO"/>
	</typeAliases>
</configuration>
```

### VO, Mapper 관련 파일 생성

**MemberVO.java** 파일 생성

```java
package com.member.dto;

public class MemberVO {

	private String userId;
	private String userPW;
	private String userName;

	public String getUserId() {
		return userId;
	}
	public void setUserId(String userId) {
		this.userId = userId;
	}
	public String getUserPW() {
		return userPW;
	}
	public void setUserPW(String userPW) {
		this.userPW = userPW;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}

	@Override
	public String toString() {
		return "MemberVO [userId=" + userId + ", userPW=" + userPW + ", userName=" + userName + "]";
	}

}
```

**MemberDAO.java** Interface 파일 생성

```java
package com.member.dao;

import java.util.List;

import org.apache.ibatis.annotations.Select;

import com.member.dto.MemberVO;

public interface MemberDAO {

	@Select("SELECT NOW()")
	public String getTime();

	public List<MemberVO> selectMember() throws Exception;

}
```

**MemberDAOImpl.java** MemberDAOImpl 파일 생성

```java
package com.member.dao;

import java.util.List;

import javax.inject.Inject;

import org.apache.ibatis.session.SqlSession;
import org.springframework.stereotype.Repository;

import com.member.dto.MemberVO;

@Repository
public class MemberDAOImpl implements MemberDAO {

	@Inject
	private SqlSession sqlSession;

	private static final String Namespace = "com.member.dao.MemberDAO.memberMapper";

	@Override
	public List<MemberVO> selectMember() throws Exception {

		return sqlSession.selectList(Namespace+".selectMember");
	}

}

```

**src/main/resources/mappers** 아래 **memberMapper.xml** 파일 생성

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.member.dao.MemberDAO.memberMapper">

    <select id="selectMember" resultType="memberVO">
        SELECT USER_ID AS ID
        	, USER_PW AS PW
        	, USER_NAME AS NAME
        FROM TB_TEST
    </select>

</mapper>
```

### Service 관련 파일 생성

**MemberService.java** 파일 생성

```java
package com.member.service;

import java.util.List;

import com.member.dto.MemberVO;

public interface MemberService {

    public List<MemberVO> selectMember() throws Exception;

}

```

**MemberServiceImpl.java** 파일 생성

```java
package com.member.service;

import java.util.List;

import javax.inject.Inject;

import org.springframework.stereotype.Service;

import com.member.dao.MemberDAO;
import com.member.dto.MemberVO;

@Service
public class MemberServiceImpl implements MemberService {

	@Inject
	private MemberDAO dao;

	@Override
	public List<MemberVO> selectMember() throws Exception {
		return dao.selectMember();
	}

}
```

### Controller 파일 생성

**HomeController.java** 파일 생성

```java
package com.member.controller;

import java.text.DateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;

import javax.inject.Inject;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.member.dto.MemberVO;
import com.member.service.MemberService;

/**
 * Handles requests for the application home page.
 */
@Controller
public class HomeController {

	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

    @Inject
    private MemberService memberService;

	/**
	 * Simply selects the home view to render by returning its name.
	 */
	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String home(Locale locale, Model model) throws Exception {
		logger.info("Welcome home! The client locale is {}.", locale);

		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

		String formattedDate = dateFormat.format(date);

		model.addAttribute("serverTime", formattedDate );

		// 멤버 조회
		List<MemberVO> memberList = memberService.selectMember();
		model.addAttribute("memberList", memberList);

		return "home";
	}

}
```

### jsp 파일 수정

**webapp\WEB-INF\views\home.jsp** 파일 수정

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!
</h1>
<P>  The time on the server is ${serverTime}. </P>
<table>
	<thead>
		<tr>
			<th>아이디</th>
			<th>비밀번호</th>
			<th>이름</th>
		</tr>
	</thead>
	<tbody>
		<c:forEach items="${memberList}" var="item">
			<tr>
				<td>${item.id}</td>
				<td>${item.pw}</td>
				<td>${item.name}</td>
			</tr>
		</c:forEach>
	</tbody>
</table>

</body>
</html>
```
