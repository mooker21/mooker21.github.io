---
title: Spring DB（PostgreSql/Oracle）연결
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
  - DB
  - PostgreSql
toc: true
toc_sticky: true
toc_label: 목차
description: Spring DB（PostgreSql/Oracle）연결
article_tag1: Spring
article_tag2: PostgreSql
article_tag3: Oracle
article_section: Spring DB（PostgreSql/Oracle）연결
meta_keywords: Spring
last_modified_at: 2023-01-18T00:00:00+08:00
---

## PostgreSql 연결

POM 설정

```xml
<dependency>
        <groupId>commons-dbcp</groupId>
        <artifactId>commons-dbcp</artifactId>
        <version>1.4</version>
    </dependency>
<dependency>
    <groupId>postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>9.1-901.jdbc4</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>
```

context-datasource.xml 설정

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="org.postgresql.Driver"/>
    <property name="url" value="jdbc:postgresql://127.0.0.1:5432/testdb"/>
    <property name="username" value="postgres" />
    <property name="password" value="postgres" />
</bean>
```

## Oracle 설정

POM 설정

```xml
<dependency>
    <groupId>ojdbc</groupId>
    <artifactId>ojdbc</artifactId>
    <version>12.2.0.1</version>
    <scope>system</scope>
    <systemPath>${basedir}/src/main/webapp/WEB-INF/lib/ojdbc8.jar</systemPath>
</dependency>
<!-- myBaitis S -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.7</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
</dependency>
<!-- myBaitis E -->
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
```

context-datasource.xml 설정

```xml
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="core.log.jdbc.driver.OracleDriver"/>
		<property name="url"             value="jdbc:oracle:thin:@192.168.0.1:1521:orcl"/>
        <property name="username" value="username"/>
        <property name="password" value="password"/>
    </bean>
```

