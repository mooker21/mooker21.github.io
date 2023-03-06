---
title: Spring IoC 의존성주입 설정 방법
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
  - Ioc
toc: true
toc_sticky: true
toc_label: 목차
description: Spring IoC 의존성주입 설정 방법
article_tag1: Spring IoC 의존성주입 설정 방법
article_tag2:
article_tag3:
article_section: Spring IoC 의존성주입 설정 방법
meta_keywords: Java, Spring, IoC
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

[Spring Annotation 설정 정리]({{ "" | relative_url }}{{% post_url 2021-06-12-Spring-Annotation-설정.md}}){:target="\_blank"}
{: .notice--info}

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

## Autowiring (자동 주입 설정 - autowire)

* Spring IoC container는 서로 관련된 Bean 객체를 자동으로 엮어줄 수 있다.
* 자동엮기(autowiring)는 각각의 bean 단위로 설정되며, 자동엮기 기능을 사용하면 <property/>나 <constructorarg/>를 지정할 필요가 없어지므로, 타이핑일 줄일 수 있다.
* 자동엮기에는 5가지 모드가 있으며, XML 기반 설정에서는 <bean/> element의 'autowire' attribute를 사용하여 설정
할 수 있다.

[Spring Annotation 설정 정리]({{ "" | relative_url }}{{% post_url 2021-06-12-Spring-Annotation-설정.md}}){:target="\_blank"}
{: .notice--info}

### 참조타입(byName), 타입(byType)

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

```xml
<bean id="more" class="com.member.service.comInformationService">
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

<bean id="moreComplexObject" class="example.ComplexObject">
	<!-- results in a setAdminEmails(java.util.Properties) call -->
	<property name="adminEmails">
		<props>
			<prop key="administrator">administrator@example.org</prop>
			<prop key="support">support@example.org</prop>
			<prop key="development">development@example.org</prop>
		</props>
	</property>
	<!-- results in a setSomeList(java.util.List) call -->
	<property name="someList">
		<list>
			<value>a list element followed by a reference</value>
			<ref bean="myDataSource" />
		</list>
	</property>
	<!-- results in a setSomeMap(java.util.Map) call -->
	<property name="someMap">
		<map>
			<entry>
				<key><value>an entry</value></key>
				<value>just some string</value>
			</entry>
			<entry>
				<key><value>a ref</value></key>
				<ref bean="myDataSource" />
			</entry>
		</map>
	</property>
	<!-- results in a setSomeSet(java.util.Set) call -->
	<property name="someSet">
		<set>
			<value>just some string</value>
			<ref bean="myDataSource" />
		</set>
	</property>
</bean>
```

### Null 주입

Java의 null 값을 사용하기 위해서 <null/> element를 사용한다. Spring IoC container는 value 값이 설정되어 있지 않은 경우 빈문자열
(“”)로 인식한다.

```xml
<bean class="ExampleBean">
	<property name="email"><value/></property>
</bean>
<bean class="ExampleBean">
	<property name="email"><null/></property>
</bean>
```

### 간단 표기

```xml
<property name="myProperty">
	<ref bean="myBean">
</property>
<!-- 아래와 같이 변경 가능 --> 
<property name="myProperty" ref="myBean"/>

<constructor-arg>
<ref bean="myBean">
</constructor-arg>
<!-- 아래와 같이 변경 가능 --> 
<constructor-arg ref="myBean"/>

<entry>
	<key>
		<ref bean="myKeyBean" />
	</key>
	<ref bean="myValueBean" />
</entry>
<!-- 아래와 같이 변경 가능 --> 
<entry key-ref="myKeyBean" value-ref="myValueBean"/>

<beans>
	<bean name="classic" class="com.example.ExampleBean">
		<property name="email" value="foo@bar.com"/>
	</bean>
</beans>
<!-- 아래와 같이 변경 가능 -->
<beans>
	<bean name="classic" class="com.example.ExampleBean" p:email="foo@bar.com"/>
</beans>

<beans>
	<bean name="john-classic" class="com.example.Person">
		<property name="name" value="John Doe"/>
		<property name="spouse" ref="jane"/>
	</bean>
	<bean name="jane" class="com.example.Person">
		<property name="name" value="Jane Doe"/>
	</bean>
</beans>
<!-- 아래와 같이 변경 가능 -->
<beans>
	<bean name="john-modern" class="com.example.Person" p:name="John Doe" p:spouse-ref="jane"/>
	<bean name="jane" class="com.example.Person">
		<property name="name" value="Jane Doe"/>
	</bean>
</beans>
```
