---
title: Spring Mybatis 개발
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
  - Mybatis
toc: true
toc_sticky: true
toc_label: 목차
description: Mybatis 개발
article_tag1: Spring
article_tag2: Mybatis
article_tag3:
article_section: Mybatis 개발
meta_keywords: Java, Spring, Mybatis
last_modified_at: 2021-06-13T00:00:00+08:00
---

국가 표준프레임워크 문서를 복사 정리 함 -> [관련 파일 Link]({{ "https://www.egovframe.go.kr/cmm/file/readDownloadFile.do?fileId=FILE_000000000016699&fileSn=5" }}){:target="\_blank"}

[myBatis 관련 자료 Link]({{ "https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#select" }}){:target="\_blank"}


## MyBatis를 활용한 Persistence Layer 개발 방법

1. **[MyBatis 설정 1] SQL Mapper XML 파일 작성 설정**
   * 실행할 SQL문과 관련 정보 설정
   * SELECT/INSERT/UPDATE/DELETE, Parameter/Result Object, Dynamic SQL 등
2. **[MyBatis 설정 2] MyBatis Configuration XML 파일 작성**
   * MyBatis 동작에 필요한 옵션을 설정
   * `<mapper>`: SQL Mapper XML 파일의 위치
3. **[스프링연동 설정] SqlSessionFactoryBean 정의**
   * Spring와 MyBatis 연동을 위한 설정
   * 역할) MyBatis 관련 메서드 실행을 위한 SqlSession 객체를 생성
   * dataSource, configLocation, mapperLocations 속성 설정
4. **DAO 클래스 작성**
   * 방법1) SqlSessionDaoSupport를 상속하는 EgovAbstractMapper 클래스를 상속받아 확장/구현
     * 실행할 SQL문을 호출하기 위한 메서드 구현: SQL Mapping XML 내에 정의한 각 Statement id를 매개변수로 전달
   * 방법2) DAO 클래스를 Interface로 작성하고, 각 Statement id와 메서드명을 동일하게 작성 (Mapper Interface 방식)
     * Annotation을 이용한 SQL문 작성 가능
     * 메서드명을 Statement id로 사용하기 때문에, 코드 최소화 가능

## MyBatis Configuration XML 파일 작성

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-
config.dtd">
<configuration>

    <typeAliases>
        <typeAlias alias="deptVO" type=“x.y.z.service.DeptVO" />
        <typeAlias alias="empVO" type="x.y.z.service.EmpVO" />
    </typeAliases>
    <mappers>
        <mapper resource ="META-INF/sqlmap/mappers/lab-dept.xml" />
        <mapper resource ="META-INF/sqlmap/mappers/lab-emp.xml" />
    </mappers>

</configuration>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<!--Mybatis 설정 -->
	<settings>
		<!-- 전통적인 데이터베이스 컬럼명 형태인 A_COLUMN을 CamelCase형태의 자바 프로퍼티명 형태인 aColumn으로 자동으로 매핑하도록 함 -->
		<setting name="mapUnderscoreToCamelCase" value="true"></setting>
		<!--  파라미터에 Null 값이 있을 경우 에러 처리 -->
		<setting name="jdbcTypeForNull" value="VARCHAR"></setting>
	</settings>
	
	<!-- Type Aliases 설정-->
	<typeAliases> 
		<typeAlias alias="egovMap" 			type="egovframework.rte.psl.dataaccess.util.EgovMap" />
		<typeAlias alias="FileVO"			type="egovframework.com.cmm.service.FileVO" />
		<typeAlias alias="ComDefaultCodeVO" type="egovframework.com.cmm.ComDefaultCodeVO" />
		<typeAlias alias="comDefaultVO"		type="egovframework.com.cmm.ComDefaultVO" />
	</typeAliases>
    
</configuration>
```

| 요소     |     설명      |
| :--------:|--------------|
| `properties`     | 설정 파일 내에서 ${key} 와 같은 형태로 외부 properties 파일을 참조할 수 있다. |
| `settings`       | 런타임시 MyBatis의 행위를 조정하기 위한 옵션 설정을 통해 최적화할 수 있도록 지원한다|
| `typeAliases`     | 타입 별칭을 통해 자바타입에 대한 좀더 짧은 이름을 사용할 수 있다. 오직 XML 설정에서만 사용되며, 타이핑을 줄이기 위해 사용 된다.|
| `typeHandlers`   | javaType과 jdbcType 일치를 위해 TypeHandler 구현체를 등록하여 사용할 수 있다. |
| `environments`   |환경에 따라 MyBatis 설정을 달리 적용할 수 있도록 지원한다.|
| `Mappers`        | 매핑할 SQL 구문이 정의된 파일을 지정한다 |


## SqlSessionFactoryBean 정의

* Spring와 MyBatis 연동을 위한 설정으로, MyBatis 관련 메서드 실행을 위해 SqlSession 객체가 필요
* 스프링에서 SqlSession 객체를 생성하고 관리할 수 있도록, SqlSessionFactoryBean을 정의
  * Id와 class는 고정값
  * dataSource : 스프링에서 설정한 Datasource Bean id를 설정하여 MyBatis가 DataSource를 사용하게 한다.
  * configLocation : MyBatis Configuration XML 파일이 위치하는 곳을 설정한다.
  * mapperLocations : SQL Mapper XML 파일을 일괄 지정할 수 있다. 단, Configuration 파일에 중복 선언할 수 없다.

```xml
<!-- SqlSession setup for MyBatis Database Layer -->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:/META-INF/sqlmap/config/sql-mapper-config.xml" />
    <!-- <property name="mapperLocations" value="classpath:/META-INF/sqlmap/mappers/*.xml" /> -->
</bean>
```

## MyBatis를 활용한 자바 클래서 작성 (Class방식)

`@Repository` 어노테이션 사용

![image](https://user-images.githubusercontent.com/83876951/223758035-45e3ca79-3955-410b-9e00-8e0601b93931.png)

## MyBatis를 활용한 자바클래스 작성 (Mapper Interface방식)

* `@Mapper` 어노테이션을 사용한다.
* 기존 DAO 클래스의 MyBatis 메서드 호출 코드를 최소화시킨 방법으로, 각 Statement id와 메서드명을 동일하게
작성하면 MyBatis가 자동으로 SQL문을 호출한다.
* 실제 내부적으로 MyBatis는 풀네임을 포함한 메서드명을 Statement id로 사용한다.

![image](https://user-images.githubusercontent.com/83876951/223759202-b8593181-7861-4916-8b9b-734c5ee01c84.png)


* 이 때 SQL Mapper XML 파일의 namespace값을 해당 Mapper의 풀네임으로 설정해야 한다.
* MyBatis는 해당 Mapper의 풀네임과 일치하는 namespace에서 메서드명과 동일한 id를 가진 Statement를 호출한다.
* namespace : 각 SQL Mapper XML을 구분

![image](https://user-images.githubusercontent.com/83876951/223758730-247e3323-9bfb-488b-aad9-f40b71d4201b.png)


* @Mapper를 사용하여 Mapper Interface가 동작하도록 하려면, MapperConfigurer 클래스를 빈으로 등록한다.
* MapperConfigurer는 @Mapper를 자동 스캔하고, MyBatis 설정의 편리함을 제공한다.
* basePackage : 스캔 대상에 포함시킬 Mapper Interface가 속한 패키지를 지정

context-mapper.xml에 다음과 같이 MapperConfigurer 설정

```xml
<!-- MapperConfigurer setup for MyBatis Database Layer -->
<bean class="egovframework.rte.psl.dataaccess.mapper.MapperConfigurer">
    <property name="basePackage" value=“x.y.z.mapper" />
</bean>
```

### Mapper Interface 방식의 어노테이션을 이용한 SQL문 작성

* 인터페이스 메소드 위에 @Statement(Select, Insert, Update, Delete …)를 선언하여 쿼리를 작성한다.
* SQL Mapper XML을 작성할 필요가 없으나, Dynamic 쿼리를 사용하지 못하고 쿼리의 유연성이 떨어진다.

```java
@Mapper("deptMapper")
public interface DeptMapper {

    @Select("select DEPT_NO as deptNo, DEPT_NAME as deptName, LOC as loc from DEPT where DEPT_NO = 
#{deptNo}")
    public DeptVO selectDept(BigDecimal deptNo);
    
    @Insert("insert into DEPT(DEPT_NO, DEPT_NAME, LOC) values (#{deptNo}, #{deptName}, #{loc})")
    public void insertDept(DeptVO vo);

    @Update("update DEPT set DEPT_NAME = #{deptName}, LOC = #{loc} WHERE DEPT_NO = #{deptNo}")
    public int updateDept(DeptVO vo);

    @Delete("delete from DEPT WHERE DEPT_NO = #{deptNo}")
    public int deleteDept(BigDecimal deptNo);
}
```
