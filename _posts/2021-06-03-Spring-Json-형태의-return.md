---
title: Spring Json 형태의 Return
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
  - Json
toc: true
toc_sticky: true
toc_label: 목차
description: Spring Json 형태의 Return
article_tag1: Java
article_tag2: Spring
article_tag3: Json
article_section: Spring Json 형태의 Return
meta_keywords:
last_modified_at: 2021-06-03T00:00:00+08:00
---

## jsonView

### pom.xml 설정

```xml
<!-- Ajax -->
<dependency>
  <groupId>net.sf.json-lib</groupId>
  <artifactId>json-lib</artifactId>
  <version>2.4</version>
  <classifier>jdk15</classifier>
</dependency>

<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-asl</artifactId>
  <version>1.8.0</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>1.8.0</version>
</dependency>
```

### dispatcherServlet.xml 설정

```xml
<!-- jsonView -->
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver" id="viewResolver" p:order="0"/>
<bean class="org.springframework.web.servlet.view.json.MappingJacksonJsonView" id="jsonView">
    <property name="contentType" value="application/json;charset=UTF-8"/>
</bean>
```

### Java 소스 코드

ModelAndView.setViewName("jsonView")를 통해 json형태 return.

```java
@RequestMapping(value="/jsonViewTest.do")
public ModelAndView jsonViewTest(@RequestParam Map<String, Object> params, HttpServletRequest request){
    ModelAndView mv = new ModelAndView();
    List<Map<String, Object>> result = new ArrayList<HashMap<String, Object>>();

    result = testService.selectData(params);
    mv.addObject("result", result);
    mv.setViewName("jsonView"); // json 지정
    return mv;
}
```

### 결과 형태

result 와 같이 warping 이 됨.

```json
{
  "result": [
    { "col1": "data1", "col2": "data2" },
    { "col1": "data3", "col2": "data4" }
  ]
}
```

## @ResponsBody 어노테이션을 이용하는 방법

### pom.xml 설정

@ResponsBody 어노테이션을 사용시 다음의 pom 설정이 필요 (Spring 4.x 버전일 경우)

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="webBindingInitializer">
        <bean class="egovframework.example.cmmn.web.EgovBindingInitializer"/>
    </property>
    <property name="messageConverters">
      <list>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter" />
        <bean class="org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter">
          <property name="supportedMediaTypes">
            <list>
              <value>*/*;charset=UTF-8</value>
            </list>
          </property>
        </bean>
      </list>
    </property>

</bean>
```

[참고]({{ "https://www.egovframe.go.kr/home/qainfo/qainfoRead.do?pagerOffset=0&searchKey=all&searchValue=Spring+4.1.2+jackson&menuNo=69&qaId=QA_00000000000016748" }}){:target="\_blank"}

### bean 설정

@ResponsBody 어노테이션을 사용시 다음의 bean 설정이 필요 (Spring 3.x 버전일 경우)

```xml
<bean id="jsonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter" />
```

### Java 소스 코드

```java
@RequestMapping(value="/responseBodyTest")
@ResponseBody
public List<Map<String, Object>> responseBodyTest(@RequestParam Map<String, Object> params, HttpServletRequest request){
    List<Map<String, Object>> result = new ArrayList<HashMap<String, Object>>();

    result = testService.selectData(params);
    return result;
}
```

### 결과형태

```json
[
  { "col1": "data1", "col2": "data2" },
  { "col1": "data3", "col2": "data4" }
]
```
