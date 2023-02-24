---
title: eGovFrame Swagger2 적용
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
  - eGovFrame
toc: true
toc_sticky: true
toc_label: 목차
description: eGovFrame Swagger2 적용
article_tag1: Python
article_tag2: tkinter
article_tag3:
article_section: eGovFrame Swagger2 적용
meta_keywords: java, spring, egove
last_modified_at: 2023-02-22T00:00:00+08:00
---

[참고]({{ "https://medium.com/@jinnyjinnyjinjin/java-spring-boot-swagger-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-4f83029bd57b" }}){:target="\_blank"} 사이트를 통해 재정리.
[에러처리참고1]({{ "https://blogdeveloperspot.blogspot.com/2018/04/maven-spring-swagger2-guava-error.html" }}){:target="\_blank"}
[에러처리참고2]({{ "https://stackoverflow.com/questions/71095560/java-lang-nosuchmethoderror-com-google-common-collect-immutablemap-error-when" }}){:target="\_blank"}

## pom.xml

```xml
<!-- START OF : swagger -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>23.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- END OF : swagger -->
```

### 빌드 에러 발생

빌드시 다음과 같은 에러 발생 발생, 하여 위의 guava dependency를 추가 함.

```console
nested exception is java.lang.NoSuchMethodError: com.google.common.collect.FluentIterable.toList()Lcom/google/common/collect/ImmutableList;
```

## eGovFrame-com-servlet.xml

```xml
<!--
<bean id="swaggerConfig" class="springfox.documentation.swagger2.configuration.Swagger2DocumentationConfiguration"/>
-->
<bean id="swaggerConfig" class="egovframework.com.cmm.config.SwaggerConfig"/>

<mvc:resources location="classpath:/META-INF/resources/" mapping="swagger-ui.html"/>
<mvc:resources location="classpath:/META-INF/resources/webjars/" mapping="/webjars/**"/>
```

## SwaggerConfig.java 파일 생성

```java
package egovframework.com.cmm.config;

import java.util.ArrayList;

import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.builders.ResponseMessageBuilder;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.ResponseMessage;
import springfox.documentation.service.VendorExtension;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@EnableWebMvc // spring-security와 연결할 때 이 부분이 없으면 404에러 발생.
@EnableSwagger2 // 기능을 활성화하는 어노테이션.
public class SwaggerConfig {
    
    @Bean
    public Docket customImplementation() {
        return new Docket(DocumentationType.SWAGGER_2)
                //.host("localhost")          //서비스하는 url주소가 다른 경우 주소 설정
                //.groupName("그룹명")
                .useDefaultResponseMessages(false)  // 기존적인 응답메시지 미사용
                .globalResponseMessage(RequestMethod.POST, getArrayList()) // 정의한 응답메시지 사용
                .apiInfo(getApiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("site.rest.web")) // Swagger 적용 패키지
                //.paths(PathSelectors.any())
                .paths(PathSelectors.ant("/restApi/**"))
                // Ignore. display internal api's
                //.paths(Predicates.not(PathSelectors.ant("/api/mail/**")))
                //.paths(Predicates.not(PathSelectors.ant("/api/**/internal/**")))
                //.paths(Predicates.not(PathSelectors.ant("/api/**/externall/**")))
                .build();
    }

    // 선택항목(Swagger UI에서 보여지는 정보)
    public ApiInfo getApiInfo() {
        return new ApiInfo(
        		"title", // title,
                "REST Api Service", // description
                "1.0", // version
                null, //"https://www.test.go.kr", // termsOfServiceUrl
                new Contact("","",""), //new Contact("테스트시스템","https://www.test.go.kr","test@test.email"), // contact
                null, //"Apache License Version 2.0", // license
                null, //"https://www.test.go.kr", // licenseUrl
                new ArrayList<VendorExtension>()); //vendorExtensions
    }

    private ArrayList<ResponseMessage> getArrayList() {
        ArrayList<ResponseMessage> lists = new ArrayList<ResponseMessage>();
        lists.add(new ResponseMessageBuilder().code(500).message("500 ERROR").build());
        lists.add(new ResponseMessageBuilder().code(403).message("403 ERROR").build());
        lists.add(new ResponseMessageBuilder().code(401).message("401 ERROR").build());
        return lists;
    }
}
```

## context-security.xml 예외 처리

```xml
<!-- swagger -->
<security:http pattern="/v2/api-docs" security="none"/>
<security:http pattern="/swagger-resources/**" security="none"/>
<security:http pattern="/webjars/**" security="none"/>
<security:http pattern="/swagger-ui.html" security="none"/>
<security:http pattern="/swagger-ui/**" security="none"/>
```

## Controller 처리 방법

```java
@Api(tags = "restApi", description = "") // Swagger 설명
@RestController
@RequestMapping("/restApi")
//@SessionAttributes(types=SessionVO.class)
public class RestApiManageController {
	
    private static final Logger LOGGER = LoggerFactory.getLogger(RestApiManageController.class);

    @Resource(name="egovMessageSource")
    EgovMessageSource egovMessageSource;
    
    @Resource(name = "restApiService")
    private RestApiService restApiService;
    
    @Resource MappingJackson2JsonView jsonView;
    
    /** EgovPropertyService */
    @Resource(name = "propertiesService")
    protected EgovPropertyService propertiesService;
        
    /**
	 * 연계정보 데이터 등록
	 * 
	 * @param HashMap
	 * @return ModelAndView
	 * @exception Exception
	 */
	@ApiOperation(value = "연계정보 등록", notes = "연계정보 등록 Api")
    @ResponseBody
    @RequestMapping(value={"/sendDngData"}, method = {RequestMethod.POST})
	public Map<String, Object> sendDngData(@RequestBody @ApiParam(value = "연계정보 등록 Model", required = true) RequestApiDngVO requestApiDngVO) {
		
		HashMap<String, Object> result = new HashMap<String, Object>();
		
		try{
    		result = restApiService.insertSemsDngData(requestApiDngVO);
        }catch (DataAccessException e) {
        	result.put("message", egovMessageSource.getMessage("site.fail.returnCode.02"));
        	result.put("retrunCode", "02");
		}catch (Exception e) {
			result.put("message", egovMessageSource.getMessage("site.fail.returnCode.01"));
        	result.put("retrunCode", "01");
		}
	
		return result;
    }
}
```

## VO 설정 방법

```java
package site.rest.service.vo;

import com.fasterxml.jackson.annotation.JsonProperty;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel(value = "연계정보 등록", description = "설명 ~~ 를 가진 Domain Class")
public class RequestApiDngVO {
	
	@ApiModelProperty(value = "아이디", required = true, example = "", dataType = "string")
	@JsonProperty("cstrnId") // Json 처리시 추가 필요
	private String cstrnId;
	
	@ApiModelProperty(value = "구분코드1", required = true, example = "0500", dataType = "string")
	@JsonProperty("gubunCd1")
	private String gubunCd1;
	
	@ApiModelProperty(value = "구분코드2", required = true, example = "0001", dataType = "string")
	@JsonProperty("gubunCd2")
	private String gubunCd2;

	@ApiModelProperty(value = "발생일시(yyyyMMddHH24MISS)", required = true, example = "20230222103645", dataType = "string")
	@JsonProperty("ocrnDt")
	private String ocrnDt;
	
	@ApiModelProperty(value = "이름", required = false, example = "홍길동", dataType = "string")
	@JsonProperty("workerNm")
	private String workerNm;
	
	@ApiModelProperty(value = "정상여부", required = false, example = "Y", dataType = "string")
	@JsonProperty("nmlYn")
	private String nmlYn;
	
    // Getter, Setter 설정 해주면 됨
}
```

