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

[참고1]({{ "https://medium.com/@jinnyjinnyjinjin/java-spring-boot-swagger-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-4f83029bd57b" }}){:target="\_blank"} [참고2]({{ "https://soo-vely-dev.tistory.com/228" }}){:target="\_blank"} 사이트를 통해 재정리.

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
<security:http pattern="/swagger-ui/**" security="none"/>
<!-- Swagger3에서는 다음과 같음 -->
<security:http pattern="/swagger-ui.html" security="none"/> 
```
라이브러리를 확인하면 사용 파일들을 확인 가능.

![image](https://user-images.githubusercontent.com/83876951/221448983-45943fe5-906d-4f72-9861-54737d5bba50.png)


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

## 기타 Controller 참고 소스

```java
package site.rest.web;

import java.nio.charset.Charset;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.dao.DataAccessException;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

import egovframework.rte.psl.dataaccess.util.EgovMap;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiModelProperty;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiResponse;
import io.swagger.annotations.ApiResponses;
import sems.rest.service.vo.RestDeviceTagDataVO;
import sems.rest.service.vo.SafeDeviceTagDataInfVO;

// rest api 스웨거 오픈을 위한 테스트 컨트롤러
//@Api(tags = "Test Example", description = "설명") // Controller에 대한 Swagger 설명
//@RestController
//@RequestMapping("/rest-api/test")
//@SessionAttributes(types=SessionVO.class)
public class RestApiTestManageController {
	
    private static final Logger LOGGER = LoggerFactory.getLogger(RestApiTestManageController.class);

    
    @ApiModelProperty(value = "유저 이름",  dataType = "string")
    private String name;

    @ApiOperation(value = "카드목록 조회", tags = "테스트", httpMethod = "GET", notes = "전체 카드목록을 조회한다.")
    @ApiImplicitParams( value = {
            @ApiImplicitParam(name = "age", value = "유저 나이", required = true, dataType = "integer", paramType = "form"),
            @ApiImplicitParam(name = "name", value = "유저 이름", required = true, dataType = "string", paramType = "form"),
            @ApiImplicitParam(name = "userNo", value = "유저 번호", required = true, dataType = "long", paramType = "form")})
    @GetMapping("/test/{cardNo}")
    public ModelAndView getTest(@PathVariable("cardNo") String cardNo, HttpServletRequest request) throws Exception {

    	LOGGER.debug("get방식 호출");

        Enumeration<String> eHeader = request.getHeaderNames();
        HashMap<String, Object> headerMap = new HashMap<String, Object>();

        while(eHeader.hasMoreElements()) {
            String headerName = eHeader.nextElement();
            headerMap.put(headerName, request.getHeader(headerName));
        }

        LOGGER.debug("cardNo" + cardNo);
        HashMap<String, Object> tempMap = new HashMap<String, Object>();

        LOGGER.debug("headerMap.get(\"content-type\")" + headerMap.get("content-type"));

        if("application/json".equals(headerMap.get("content-type"))){
            //JSONObject jsonObject = JsonUtil.getStringToJsonObecjt(request);
        }

        StringBuffer requestURL = request.getRequestURL();
        String servletPath = request.getServletPath();

        ModelAndView mav = new ModelAndView("jsonView");
        EgovMap resultMap = new EgovMap();

        try {
              // 목록 조회
              // mav.addObject("dataList" , defaultService.select(tempMap));
              // DB SQL 실행 테스트
               resultMap.put("result" , "SUCCESS");
               resultMap.put("message" , "정상적으로 조회되었습니다.");
           } catch(Exception ex) {
               resultMap.put("result" , "ERROR");
               resultMap.put("message" , "조회중 오류가 발생하였습니다");
               ex.printStackTrace();
           } finally {
               mav.addObject("result" , resultMap);
           }

           return mav;

    }

    @PostMapping("/test/{cardNo}")
    public ModelAndView postTest(@PathVariable("cardNo") String cardNo, HttpServletRequest request) throws Exception {
    	LOGGER.debug("post방식 호출" + cardNo);
        ModelAndView mav = new ModelAndView("jsonView");
        EgovMap resultMap = new EgovMap();
        resultMap.put("result" , "SUCCESS");
        resultMap.put("message" , "정상적으로 조회되었습니다.");
        mav.addObject("result" , resultMap);
        return mav;
    }

    @PutMapping("/test/{cardNo}")
    public ModelAndView putTest(@PathVariable("cardNo") String cardNo, HttpServletRequest request) throws Exception {
    	LOGGER.debug("put방식 호출" + cardNo);
        ModelAndView mav = new ModelAndView("jsonView");
        EgovMap resultMap = new EgovMap();
        resultMap.put("result" , "SUCCESS");
        resultMap.put("message" , "정상적으로 조회되었습니다.");
        mav.addObject("result" , resultMap);
        return mav;
    }

    @DeleteMapping("/test/{cardNo}")
    public ModelAndView deleteTest(@PathVariable("cardNo") String cardNo, HttpServletRequest request) throws Exception {
    	LOGGER.debug("delete방식 호출" + cardNo);
        ModelAndView mav = new ModelAndView("jsonView");
        EgovMap resultMap = new EgovMap();
        resultMap.put("result" , "SUCCESS");
        resultMap.put("message" , "정상적으로 조회되었습니다.");
        mav.addObject("result" , resultMap);
        return mav;
    }
    
    @ApiOperation(  // API에 대한 Swagger 설명
            value="서비스",
            notes = "서비스입니다.",
            httpMethod = "POST",
            consumes = "application/json",
            produces = "application/json",
            protocols = "http",
            responseHeaders = {
                    //headers
            })
    @ApiResponses({  // Response Message에 대한 Swagger 설명
            @ApiResponse(code = 200, message = "OK"),
            @ApiResponse(code = 404, message = "No params")
    })
    @PostMapping(value="/service", consumes = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<HashMap<String, Object>> service(HttpServletRequest Header,
                                                       @RequestBody HashMap<String, Object> inputVO) {

        //return callService.syncCall(Header, inputVO);
        HashMap<String, Object> result = new HashMap<String, Object>();
		result.put("message", "fail.common.insert");
    	result.put("status", "fail");
	
        HttpHeaders header = new HttpHeaders();
        header.setContentType(new MediaType("application", "json", Charset.forName("UTF-8")));
        
		return  new ResponseEntity<>(result, header, HttpStatus.OK);
    }

    @ApiImplicitParams({
            @ApiImplicitParam(name = "area", value = "지역", required = true, dataType = "String", paramType = "path"),
            @ApiImplicitParam(name = "param1", value = "파라미터1", required = true, dataType = "String", paramType = "query"),
            @ApiImplicitParam(name = "param2", value = "파마미터2", required = false, dataType = "int", paramType = "query")
    })
    
    @GetMapping(value = "/home/{area}")
    public String home(@PathVariable String area, @RequestParam String param1, @RequestParam int param2) {
        return "home";
    }
    
    @GetMapping(value = "/")
    public String home(Model model) {
        model.addAttribute("hi", "Hello~~");
        return "home";
    }
    
}
```
