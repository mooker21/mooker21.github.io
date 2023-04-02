---
title: Java HashMap → VO 로 변경
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
  - HashMap
toc: true
toc_sticky: true
toc_label: 목차
description: Java HashMap → VO 로 변경
article_tag1: Java
article_tag2: Spring
article_tag3: HashMap
article_section: Java HashMap → VO 로 변경
meta_keywords: java, spring, hashmap
last_modified_at: 2023-03-19T00:00:00+08:00
---

## Java HashMap → VO 로 변경

```java
import java.util.HashMap;
import javax.annotation.Resource;
import org.springframework.stereotype.Service;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.MapperFeature;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
	* 기상청 API를 호출하여 날씨 정보를 입력한다.
	* @param
	* @return
	* @throws Exception
	*/
public void getWtrInfo() throws Exception {
	
	// 날씨정보
	HashMap<String, Object> weatherInfo = WeatherUtil.getWeatherInfo("60", "127");
	
	ObjectMapper mapper = new ObjectMapper().configure(MapperFeature.ACCEPT_CASE_INSENSITIVE_PROPERTIES, true);
	WtrVO webInfo = mapper.convertValue(weatherInfo, new TypeReference<WtrVO>() {});
	
	// 날씨 정보의 날짜,시간이 존재시 등록 처리
	if (!StringUtil.isEmpty(webInfo.getBaseDate()) && !StringUtil.isEmpty(webInfo.getBaseTime())) {
		egovWtrService.insertWtrInfo(webInfo); // 기본 정보 등록
		egovWtrService.reUpdateWtrInfo(webInfo); // API 호출시 전달되지 않는 날씨정보가 (자주)존재하니, null값들은 이전 값으로 update 처리
	}
}
```
