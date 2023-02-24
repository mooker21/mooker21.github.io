---
title: eGovFrame Codegen 사용
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
  - Codegen
toc: true
toc_sticky: true
toc_label: 목차
description: eGovFrame Codegen 사용
article_tag1: Java
article_tag2: Spring
article_tag3: eGovFrame
article_section: eGovFrame Codegen 사용
meta_keywords: java, spring, egoveframe, godegen
last_modified_at: 2023-02-15T00:00:00+08:00
---

## 설치 및 상세내용

[Link]({{ "https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev2:imp:codegen&s[]=codegen" }}){:target="\_blank"}

## 사용시 주의점

eGove Perspective 환경에서 실행해야 Package Selection에서 Package 정보가 보임.

![image](https://user-images.githubusercontent.com/83876951/220492392-640c3eca-3543-4768-9db2-e979b894a25f.png)
*eGove Perspective 환경*

![image](https://user-images.githubusercontent.com/83876951/220492575-61950743-9d8e-4f92-b0ea-046955a6d3b7.png)
*Template 실행*

![image](https://user-images.githubusercontent.com/83876951/220493504-5d55c06c-0467-4701-a3b7-0cf2a54fa52c.png)
*Package Selection 선택*

![image](https://user-images.githubusercontent.com/83876951/220492694-23577720-4ac6-4d87-884a-7a4d8c915e01.png)
*Package Selection에 내용이 나옴*

## CRUD Generation 내용

Generation시 DB 테이블을 temp로 생성하여 만들고, generation 이 후 테이블을 drop 하는 방식이 편함.

### Creater

1. Author: 이름
2. Create Date : 2023.02.15

### DataAcess

1. Resource(SQLMap) Foler : /site/src/main/resources/egovframework/mapper/site/stats
2. Resource(Mapper) Foler : /site/src/main/resources/egovframework/mapper/site/stats --> 이건 하지 말기 /site/src/main/java/site/mappers/stats
3. DAO Package: site.stats.service.impl
4. Mapper Package: site.mappers.stats
5. VO Package: site.stats.service.vo

### Service

1. Service Package: site.stats.service
2. Service Impl Package: site.stats.service.impl

### Web

1. Controller Package: site.stats.web
2. JSP Floder : /site/src/main/webapp/WEB-INF/jsp/site/stats

