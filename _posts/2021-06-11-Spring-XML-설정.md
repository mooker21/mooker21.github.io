---
title: Spring XML 설정 방법
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
description: Spring XML 설정 방법
article_tag1: Spring XML 설정 방법
article_tag2:
article_tag3:
article_section: Spring XML 설정 방법
meta_keywords: Java, Spring
last_modified_at: 2021-06-11T00:00:00+08:00
---

[참고]({{ "https://atoz-develop.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-XML-%EC%84%A4%EC%A0%95-%ED%8C%8C%EC%9D%BC-%EC%9E%91%EC%84%B1-%EB%B0%A9%EB%B2%95-%EC%A0%95%EB%A6%AC" }}){:target="\_blank"} 사이트를 통해 복사 정리.

## 스프링 XML 설정 파일 포맷

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

anotation 설정을 사용하기 위한 포맷

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
</beans>
```

## bean 설정 옵션

```xml
<bean id="studentDao" class="com.member.dao.StarterDao" />
```

- id : 빈 이름(id) 설정 (id를 통해 참조할 경우가 있는 경우에만 설정)
- class : 빈 타입 설정 (필수)
- scope : 빈의 scope 설정. singleton/prototype
- primary : true를 지정하여 같은 타입의 빈이 여러개 일때 우선적으로 사용할 빈 설정
- lazy-init : true를 지정하여 빈을 사용할 때 객체가 생성되도록 설정
- init-method : 빈 객체가 생성될때 호출할 메소드 설정
- destroy-method : 빈 객체가 소멸될때 호출할 메소드 설정

## 자동 주입 설정 - autowire 속성

참조타입(byName), 타입(byType)
getter, setter 설정 필요

```xml
<bean id='obj2' class='com.study.beans.TestBean1' autowire="byName/"/>
<bean id='obj3' class='com.study.beans.TestBean2' autowire="byType"/>
```

```java
public class TestBean1 {

	private DataBean1 data1;
	private DataBean1 data2;

	public DataBean1 getData1() {
		return data1;
	}
	public void setData1(DataBean1 data1) {
		this.data1 = data1;
	}
	public DataBean1 getData2() {
		return data2;
	}
	public void setData2(DataBean1 data2) {
		this.data2 = data2;
	}
}
```

### 생성자 주입

```xml
<bean id='obj7' class='com.study.beans.TestBean3' autowire="constructor">
	<constructor-arg value='200' index='0' type='int'/>
	<constructor-arg value='문자열2' index='1'/>
</bean>
```

```java
public class TestBean3 {

	private int data1;
	private String data2;
	private DataBean2 data3;
	private DataBean2 data4;

	public TestBean3(int data1, String data2, DataBean2 data3, DataBean2 data4) {
		this.data1 = data1;
		this.data2 = data2;
		this.data3 = data3;
		this.data4 = data4;
	}

	public int getData1() {
		return data1;
	}
	public void setData1(int data1) {
		this.data1 = data1;
	}
	public String getData2() {
		return data2;
	}
	public void setData2(String data2) {
		this.data2 = data2;
	}
	public DataBean2 getData3() {
		return data3;
	}
	public void setData3(DataBean2 data3) {
		this.data3 = data3;
	}
	public DataBean2 getData4() {
		return data4;
	}
	public void setData4(DataBean2 data4) {
		this.data4 = data4;
	}
}
```

### 외부라이브러리 생성자 주입 Sample

```xml
<!-- sample Util 설정 -->
<bean id="sampleUtil" class="common.serivce.SampleUtil" autowire="constructor">
	<constructor-arg value='sample.com' index='0' />  <!-- domainName -->
	<constructor-arg value='123.213.123.123' index='1' /> <!-- Ip -->
	<constructor-arg value='9003' index='2' type='int' /> <!-- Port -->
	<constructor-arg value='GOODMORNING' index='3' /> <!-- Type -->
</bean>
```

```java
public class SampleUtil {

	private static final Logger LOGGER = LoggerFactory.getLogger(SampleUtil.class);

	private ExternalLib externalLib = null;
	private String domainName; 	// 도메인명
	private String ip; 			// IP
	private int port; 			// IP Port
	private String type;		// 형태
	private String envMode;		// 환경 모드 (local, dev, real)

	/** 생성자
	 * @param domainName 도메인명
	 * @param ip 첫번째 IP
	 * @param port IP Port
	 * @throws CommException
	 */
	public SampleUtil(String domainName, String ip, int port, String type) throws CommException {
		super();
		this.domainName = domainName;
		this.ip = ip;
		this.port = port;
		this.Type = Type;
		this.envMode = System.getProperty("spring.profiles.active"); // 현재 서버 환경

		try {
			if("dev".equals(this.envMode) || "real".equals(this.envMode)) {

				// 외부 라이브러리 호출
				externalLib = ExternalLib.getSampleInstUp(this.domainName, this.ip, this.port);

			}
		} catch (Exception e) {
			throw new CommException(e);
		}
	}

	public String getDomainName() {
		return domainName;
	}

	public void setDomainName(String domainName) {
		this.domainName = domainName;
	}

	public String getIp() {
		return ip;
	}

	public void setIp(String ip) {
		this.ip = ip;
	}

	public int getPort() {
		return port;
	}

	public void setPort(int port) {
		this.port = port;
	}

	public String getType() {
		return type;
	}

	public void SetType(String type) {
		this.type = type;
	}

}
```

## DI(Dependency Injection) 설정

### 생성자 주입

```xml
<bean id="studentDao" class="com.member.dao.StudentDao" ></bean>

<bean id="registerService" class="com.member.service.StudentRegisterService">
    <constructor-arg ref="studentDao" />
</bean>
```

```java
public class StudentRegisterService {

	private StudentDao studentDao;

	public StudentRegisterService(StudentDao studentDao) {
		this.studentDao = studentDao;
	}
}
```

생성자가 여러 개인 경우

```xml
<bean id="obj2" class="com.study.beans.TestBean">
	<constructor-arg value="44" type="int" index="0"/>
	<constructor-arg value="44.44" type="double" index="1"/>
	<constructor-arg value="안녕하세요" type="java.lang.String" index="2"/>
</bean>
```

```java
public class TestBean {
	private int data1;
	private double data2;
	private String data3;

	public TestBean(int data1, double data2, String data3) {
		this.data1 = data1;
		this.data2 = data2;
		this.data3 = data3;
	}
}
```

### 프로퍼티 주입

```xml
<bean id="dataBaseConnectionInfoDev" class="com.member.DataBaseConnectionInfo">
	<property name="jdbcUrl" value="jdbc:oracle:thin:@localhost:1521:xe" />
	<property name="userId" value="scott" />
	<property name="userPw" value="tiger" />
</bean>
```

```java
public class DataBaseConnectionInfo {

    private String jdbcUrl;
    private String userId;
    private String userPw;

    public String getJdbcUrl() {
        return jdbcUrl;
    }

    public void setJdbcUrl(String jdbcUrl) {
        this.jdbcUrl = jdbcUrl;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getUserPw() {
        return userPw;
    }

    public void setUserPw(String userPw) {
        this.userPw = userPw;
    }
}
```

### 프로퍼티 주입 - List 타입

```xml
<bean id="informationService" class="com.member.service.comInformationService">
	<property name="developers">
		<list>
			<value>Cheney.</value>
			<value>Eloy.</value>
			<value>Jasper.</value>
			<value>Dillon.</value>
			<value>Kian.</value>
		</list>
	</property>
</bean>
```

```java
public class comInformationService {

	private List<String> developers;

	public List<String> getDevelopers() {
		return developers;
	}

	public void setDevelopers(List<String> developers) {
		this.developers = developers;
	}
}
```

type 설정

```xml
<property name="list2">
	<list>
		<value type='int'>100</value>
		<value type='int'>200</value>
		<value type='int'>300</value>
	</list>
</property>
```
