---
title: Spring AOP, logging
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
  - AOP
  - logging
toc: true
toc_sticky: true
toc_label: 목차
description: Spring AOP, logging
article_tag1: Spring
article_tag2: AOP
article_tag3: logging
article_section: Spring, AOP, logging
meta_keywords: Spring, AOP, logging
last_modified_at: 2021-06-06T00:00:00+08:00
---

## 주요 개념

* Join Point
  * 횡단 관심(Crosscutting Concerns) 모듈이 삽입되어 동작할 수 있는 실행 가능한 특정 위치를 말함
  * **메소드 호출, 메소드 실행 자체, 클래스 초기화, 객체 생성 시점 등**
* Pointcut
  * Pointcut은 어떤 클래스의 어느 JoinPoint를 사용할 것인지를 결정하는 선택 기능을 말함
  * **가장 일반적인 Pointcut은 ‘특정 클래스에 있는 모든 메소드 호출’로 구성된다.**
* 애스펙트(Aspect)
  * 어플리케이션이 가지고 있어야 할 로직과 그것을 실행해야 하는 지점을 정의한 것
  * **Advice와 Pointcut의 조합**
* Advice
  * Advice는 관점(Aspect)의 실제 구현체로 결합점에 삽입되어 동작할 수 있는 코드이다
  * Advice 는 결합점(JoinPoint)과 결합하여 동작하는 시점에 따라 before advice, after advice, around advice 타입으로 구분된다
  * **특정 Join point에 실행하는 코드**
* Weaving
  * Pointcut에 의해서 결정된 JoinPoint에 지정된 Advice를 삽입하는 과정
  * Weaving은 AOP가 기존의 Core Concerns 모듈의 코드에 전혀 영향을 주지 않으면서 필요한 Crosscutting Concerns 기능을 추 가할 수 있게 해주는 핵심적인 처리 과정임

## Spring의 AOP 지원

* 스프링은 프록시 기반의 런타임 Weaving 방식을 지원. (Waving 방식에는 컴파일시, JVM 클래스 로딩시, 런타임 시 엮기 방식이 있음.)
* 스프링은 AOP 구현을 위해 다음 세가지 방식을 제공.
  * @AspectJ 어노테이션을 이용한 AOP 구현
  * XML Schema를 이용한 AOP 구현
  * 스프링 API를 이용한 AOP 구현
* 국가 표준프레임워크 실행환경은 XML Schema를 이용한 AOP 구현 방법을 사용.

## Advice 결합점 결합 타입

* `Before advice`: joinpoint 전에 수행되는 advice
* `After returning advice`: joinpoint가 성공적으로 리턴된 후에 동작하는 advice
* `After throwing advice`: 예외가 발생하여 joinpoint가 빠져나갈때 수행되는 advice
* `After advice`: join point를 빠져나가는(정상적이거나 예외적인 반환) 방법에 상관없이 수행되는 advice
* `Around advice`: joinpoint 전, 후에 수행되는 advice

## Aspect 실행 순서

### 정상의 경우

1. before
2. around (대상 메소드 수행 전)
3. 대상 메소드
4. after-returning
5. after(finally)
6. around (대상 메소드 수행 후)

### 예외 발생의 경우

1. before
2. around (대상 메소드 수행 전)
3. 대상 메소드 (ArithmeticException 예외가 발생한다)
4. ~~after-returning~~
4. afterThrowing
5. after(finally)
6. ~~around (대상 메소드 수행 후)~~


## Aspect 정의하기

![image](https://user-images.githubusercontent.com/83876951/223164602-4a28a36d-4f30-4855-933d-237e11ab6ca9.png)

## [참고] context-aspect.xml 샘플

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
						http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<aop:config>
		<aop:pointcut id="egov.serviceMethod" expression="execution(* egovframework.com..impl.*Impl.*(..))" />

		<aop:aspect ref="egov.exceptionTransfer">
			<aop:after-throwing throwing="exception" pointcut-ref="egov.serviceMethod" method="transfer" />
		</aop:aspect>
	</aop:config>
	
	<bean id="egov.exceptionTransfer" class="egovframework.rte.fdl.cmmn.aspect.ExceptionTransfer">
		<property name="exceptionHandlerService">
			<list>
				<ref bean="defaultExceptionHandleManager" />
				<ref bean="otherExceptionHandleManager" />
			</list>
		</property>
	</bean>

	<bean id="defaultExceptionHandleManager" class="egovframework.rte.fdl.cmmn.exception.manager.DefaultExceptionHandleManager">
		<property name="reqExpMatcher">
			<ref bean="egov.antPathMater"/>
		</property>
		<property name="patterns">
			<list>
				<value>**service.impl.*</value>
			</list>
		</property>
		<property name="handlers">
			<list>
				<ref bean="egovHandler" />
			</list>
		</property>
	</bean>

	<bean id="otherExceptionHandleManager" class="egovframework.rte.fdl.cmmn.exception.manager.DefaultExceptionHandleManager">
		<property name="reqExpMatcher">
			<ref bean="egov.antPathMater"/>
		</property>
		<property name="patterns">
			<list>
				<value>**service.impl.*</value>
			</list>
		</property>
		<property name="handlers">
			<list>
				<ref bean="otherHandler" />
			</list>
		</property>
	</bean>
	
	<bean id="egovHandler" class="egovframework.com.cmm.EgovComExcepHndlr" />
	<bean id="otherHandler" class="egovframework.com.cmm.EgovComOthersExcepHndlr" />	
	
	<!--  Privacy Log Aspect -->
	<bean id="privacyLog" class="egovframework.com.sym.log.plg.service.EgovPrivacyLogAspect">
		<property name="maxListCount" value="1" />
		<property name="target">
			<map>
                <entry key="mberNm" value="회원명"/>
                <entry key="moblphonNo" value="휴대폰번호"/>
                <entry key="emailAdres" value="이메일"/>
                <entry key="email" value="이메일"/>
                <entry key="orgnztNm" value="조직명"/>
			</map>
		</property>
	</bean>

	<aop:config>
		<aop:aspect id="privacyLogAspect" ref="privacyLog">
			<!--  service Method -->
			<aop:after-returning returning="returnVal" 
					pointcut="execution(public * egovframework.com..impl.*ImplprivacyLog.*(..)) 
				 &amp;&amp; ! execution(public * egovframework.com.sym.log.plg.service.impl.EgovPrivacyLogServiceImpl.*(..))
				 &amp;&amp; ! execution(public * egovframework.com.sym.log.lgm.service.impl.EgovSysLogServiceImpl.*(..))
				 &amp;&amp; ! execution(public * egovframework.com.sym.log.wlg.service.impl.EgovWebLogServiceImpl.*(..))
				 "
				 method="insertLog" />
			</aop:aspect>
			<!--  warning : 자체 서비스(EgovPrivacyLogServiceImpl) 및 내부 호출 서비스, 로그 처리 부분  제외 필요 -->
	</aop:config>
	<!--  Privacy Log Aspect -->
	
	<!--  System Log Aspect -->
	<bean id="syslog" class="egovframework.com.sym.log.lgm.service.EgovSysLogAspect" />
 
	<aop:config> 
		<aop:aspect id="sysLogAspect" ref="syslog">  
			<!--  insert로 시작되는 service Method -->
			<aop:around pointcut="!execution(public * site.service.impl.RestApiServiceImpl.*(..)) 
			and (execution(public * egovframework.com..impl.*Impl.insert*(..))  or execution(public * sems..impl.*Impl.insert*(..)))" method="logInsert" />
			<!--  update로 시작되는 service Method -->
			<aop:around pointcut="!execution(public * site.service.impl.RestApiServiceImpl.*(..))
			and (execution(public * egovframework.com..impl.*Impl.update*(..)) or execution(public * sems..impl.*Impl.update*(..)))" method="logUpdate" />
			<!--  delete로 시작되는 service Method -->
			<aop:around pointcut="!execution(public * site.service.impl.RestApiServiceImpl.*(..)) 
			and (execution(public * egovframework.com..impl.*Impl.delete*(..)) or execution(public * sems..impl.*Impl.delete*(..)))" method="logDelete" />
			<!--  select로 시작되는 service Method -->
			<aop:around pointcut="!execution(public * site.service.impl.RestApiServiceImpl.*(..))
			and (execution(public * egovframework.com..impl.*Impl.select*(..)) or execution(public * sems..impl.*Impl.select*(..)))" method="logSelect" /> 
		</aop:aspect>
	</aop:config>
	<!--  System Log Aspect -->
	
	
	
	<!--  RSAKeyGeneral Aspect -->
	<bean id="RSAKeyGeneral" class="sems.cmm.aspect.RSAKeyAspect" />

	<aop:config>
		<aop:aspect id="RSAKeyAspect" ref="RSAKeyGeneral">
			<!--  annotation RSAKeyGeneral 선언된 Method -->
			<aop:after pointcut="@annotation(sems.cmm.annotation.RSAKeyGeneral)" method="keyGeneral1" />
		</aop:aspect>
	</aop:config>
	<!--  RSAKeyGeneral Aspect -->
</beans>
```
