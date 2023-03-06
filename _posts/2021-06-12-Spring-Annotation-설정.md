---
title: Spring Annotation 설정
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
  - Annotation
toc: true
toc_sticky: true
toc_label: 목차
description: Spring Annotation 설정
article_tag1: Spring Annotation 설정
article_tag2:
article_tag3:
article_section: Spring Annotation 설정
meta_keywords: Java, Spring, IoC, Annotation
last_modified_at: 2021-06-12T00:00:00+08:00
---

전자정품 표준프레임웍크 문서를 복사 정리 함

## @Required

Required annotation은 setter 메소드에 적용된다. @Required annotation이 설정된 property는 <property/>, <constructor-arg/> element를 통해서 명시적으로 값이 설정되거나, autowiring에 의해서 값이 설정되어야 한다.


```java
public class SimpleMovieLister {
	private MovieFinder movieFinder;
	@Required
	public void setMovieFinder(MovieFinder movieFinder) {
	this.movieFinder = movieFinder;
	}
	// ...
}
```

## @Autowired

Autowired annotation은 자동으로 엮을 property를 지정하기 위해 사용한다. setter 메소드, 일반적인 메소드, 생성자, field 등에 적용된다.

Setter 메소드

```java
public class SimpleMovieLister {
	private MovieFinder movieFinder;

	@Autowired
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}
	// ...
}
```

일반적인 메소드

```java
public class MovieRecommender {
	private MovieCatalog movieCatalog;
	private CustomerPreferenceDao customerPreferenceDao;

	@Autowired
	public void prepare(MovieCatalog movieCatalog, CustomerPreferenceDao customerPreferenceDao) {
		this.movieCatalog = movieCatalog;
		this.customerPreferenceDao = customerPreferenceDao;
	}
// ...
}
```

생성자 및 field

```java
public class MovieRecommender {
	@Autowired
	private MovieCatalog movieCatalog;
	private CustomerPreferenceDao customerPreferenceDao;
	
	@Autowired
	public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
		this.customerPreferenceDao = customerPreferenceDao;
	}
	// ...
}
```

## @Qualifier(1/3)

@Autowired annotation만을 사용하는 경우, 같은 Type의 Bean이 둘 이상 존재할 때 문제가 발생한다. 이를 방지하기 위해서
@Qualifier annotation을 사용하여 찾을 Bean의 대상 집합을 좁힐 수 있다. @Qualifier annotation은 field 뿐 아니라 생성자 또는
메소드의 parameter에도 사용할 수 있다.

Field

```java
public class MovieRecommender {
	@Autowired
	@Qualifier("main")
	private MovieCatalog movieCatalog;
	// ...
}
```

메소드 Parameter

```java
public class MovieRecommender {
	private MovieCatalog movieCatalog;
	private CustomerPreferenceDao customerPreferenceDao;
	@Autowired
	public void prepare(@Qualifier("main") MovieCatalog movieCatalog, CustomerPreferenceDao customerPreferenceDao) {
		this.movieCatalog = movieCatalog;
		this.customerPreferenceDao = customerPreferenceDao;
}
// ...
}
```

@Qualifier annotation의 값으로 사용되는 qualifier는 <bean/> element의 <qualifier/> element로 설정한다

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<context:annotation-config/>
	<bean class="example.SimpleMovieCatalog">
		<qualifier value="main"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean class="example.SimpleMovieCatalog">
		<qualifier value="action"/>
		<!-- inject any dependencies required by this bean -->
	</bean>

	<bean id="movieRecommender" class="example.MovieRecommender"/>
</beans
```

## @Resource

@Autowaired 와 거의 비슷함. 다른점은 이름은 지정해야 함.

@Resource annotation의 name 값으로 대상 bean을 찾을 수 있다. @Resource annotation은 field 또는 메소드에 사용할 수 있다.

```java
public class SimpleMovieLister {
	private MovieFinder movieFinder;

	@Resource(name="myMovieFinder")
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}
}
```

@Resource annotation에 name 값이 없을 경우, field 명 또는 메소드 명을 이용하여 대상 bean을 찾는다.

```java
public class SimpleMovieLister {
	private MovieFinder movieFinder;
	
	@Resource
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}
}
```

## @PostConstruct & @PreDestroy

@PostConstruct와 @PreDestroy는 각각 Instantiation callback, Destruction callback 메소드를 지정하기 위해 사용한다.

```java
public class CachingMovieLister {
	@PostConstruct
	public void populateMovieCache() {
		// populates the movie cache upon initialization...
	}
	@PreDestroy
	public void clearMovieCache() {
		// clears the movie cache upon destruction...
	}
}
```

## Auto-detecting components

Spring은 @Repository, @Service, @Controller annotation을 사용하여, 각각 Persistence, Service, Presentation 레이어의 컴포넌트로 지정하여 특별한 관리 기능을 제공하고 있다. 

@Repository, @Service, @Controller는 @Component annotation을 상속받고
있다. Spring IoC Container는 @Component annotation(또는 자손)으로 지정된 class를 XML Bean 정의 없이 자동으로 찾을 수 있다.

<context:component-scan/> element의 ‘base-package’ attribute는 컴포넌트를 찾을 기본 package이다. ‘,’를 사용하여 다수의 기본 package를 지정할 수 있다.

<conext:component-scan/>을 쓰는 경우에는 <context:annotation-config/>를 별도로 쓰지 않아도 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-4.0.xsd">

<context:component-scan base-package="org.example"/>

</beans>
```


이름 설정 @Component, @Repository, @Service, @Controller annotation의 name 값으로 bean의 이름을 지정할 수 있다. 

아래 예제의 Bean 이름은 “myMovieLister”이다

```java
@Service("myMovieLister")
public class SimpleMovieLister {
// ...
}
```

만약 name 값을 지정하지 않으면, class 이름의 첫문자를 소문자로 변환하여 Bean 이름을 자동으로 생성한다.

아래 예제의 Bean 이름은 “movieFinderImpl”이다.

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
// ...
}
```

• Scope 설정

@Scope annotation을 사용하여, 자동으로 찾은 Bean의 scope를 설정할 수 있다.

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
// ...
}
```

• Qualifier 설정

@Qualifier annotation을 사용하여, 자동으로 찾은 Bean의 qualifier를 설정할 수 있다.

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog 
```

## ApplicationContext for web application

* Spring은 Web Application에서 ApplicationContext를 쉽게 사용할 수 있도록 각종 class들을 제공하고 있다.
* Servlet 2.4 이상
  * Servlet 2.4 specification부터 Listener를 사용할 수 있다. Listener를 사용하여 ApplicationContext를 설정하기 위해서 web.xml 파
일에 다음 설정을 추가한다.
  * contextParam의 ‘contextConfigLocation’ 값은 Spring XML 설정 파일의 위치를 나타낸다.

```xml
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
