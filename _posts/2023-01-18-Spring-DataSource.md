---
title: Spring DataSorce
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
  - DataSource
toc: true
toc_sticky: true
toc_label: 목차
description: Spring DataSorce
article_tag1: Spring
article_tag2: DB
article_tag3: DataSource
article_section: Spring DataSorce
meta_keywords: Java, Spring, DB, DataSource
last_modified_at: 2023-01-18T00:00:00+08:00
---

## dataSource

```xml
<beanid="dataSource"class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <propertyname="driverClassName" value="${driver}"/>
    <propertyname="url" value="${dburl}"/>
    <propertyname="username" value="${username}"/>
    <propertyname="password" value="${password}"/>
    <propertyname="initialSize" value="0"/>
    <propertyname="maxTotal" value="8"/>
    <propertyname="maxIdle" value="8"/>
    <propertyname="minIdle" value="0"/>
    <propertyname="maxWaitMillis "value="-1"/>
</bean>
```

| 값          | 설명                                                         |
| --------  | ------------------------------------------------------------ |
| [initialPoolSize](#)    | 최초로 getConnection()Method를 통해 커넥션 풀에 채워 넣을 커넥션 개수
(maxTotal >=initialSize) |
| [maxTotal](#)    | 동시에 사용할 수 있는 최대 커넥션 개수로 기본값은 8
*CommonsDBCP1.x에서는 maxActive 로 변경 |
| [maxIdle](#)    | ConnectionPool에 반납할 때 최대로 유지될 수 있는 연결 개수로 기본값은 8
(maxTotal = maxIdle) |
| [minIdle](#) | ConnectionPool에서 최소한으로 유지할 커넥션 개수로 기본값은 0
(maxIdle >= minIdle) |
| [maxWaitMillis](#) | 커넥션 풀에 연결 가능한 커넥션이 없을 경우 반납되는 커넥션을 대기하는 시간(밀리초)이며 기본값은 무한정(응답올때까지 대기)
*CommonsDBCP1.x에서는 maxWait 로 변경 |

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

