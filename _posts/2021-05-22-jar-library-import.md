---
title: Eclipse에서 외부 jar 파일 추가 하기
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Java
toc: true
toc_sticky: true
toc_label: 목차
description: Eclipse에서 외부 jar 파일 추가 하기
article_tag1: Eclipse에서 외부 jar 파일 추가 하기
article_tag2:
article_tag3:
article_section: Eclipse에서 외부 jar 파일 추가 하기
meta_keywords: Java, Eclipse, jar
last_modified_at: 2021-05-22T00:00:00+08:00
---

# Eclipse에서 외부 jar 파일 추가 하기

1. src/main/webapp/WEB-INF 폴더에 lib 이름의 폴더 생성

2. WEB-INF/lib 폴더에 jar 파일 추가

3. 추가한 jar 파일을 선택 후 Build Path > Add to Build Path 클릭해서 Referenced Libraries 폴더에 들어가는지 확인

4. mvn clean package 또는 debug on Server 등의 메뉴를 실행 후 정상 동작 확인

5. mvn package 시에는 target/project/WEB-INF/lib 폴더에 jar 파일이 추가 되었고 war 파일에도 정상적으로 포함되었는지 확인

위와 같은 순서로 확인하면 잘 포함이 된 것임.

![image](https://user-images.githubusercontent.com/83876951/119850169-3ef92d00-bf48-11eb-8d77-569092afae7c.png)
![image](https://user-images.githubusercontent.com/83876951/119850189-46b8d180-bf48-11eb-9288-c3540dbbb96a.png)
