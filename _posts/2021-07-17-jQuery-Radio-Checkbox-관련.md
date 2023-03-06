---
title: jQuery Radio Checkbox 관련
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Javascript
toc: true
toc_sticky: true
toc_label: 목차
description: jQuery Radio Checkbox 관련
article_tag1: Javascript
article_tag2:
article_tag3:
article_section: jQuery Radio Checkbox 관련
meta_keywords: Javascript
last_modified_at: 2021-07-17T00:00:00+08:00
---

[참고]({{ "https://djkeh.github.io/articles/Client-ip-over-load-balancer-and-its-access-log-kor/" }}){:target="\_blank"} 사이트를 통해 재정리.

## jQuery Radio

### Radio 선택 값 확인

```javascript
var grpCd1 = $("input:radio[name='grpCd']:checked").val();

var grpCd2 = $("input[name='grpCd']:checked").val();

var id = $('input[name="grpCd"]:checked').attr("id");

/* 전체 Unchecked Values */
var str = "";
$('input[name="grpCd"]:not(:checked)').each(function () {
  var id = $(this).attr("id");
  var value = $(this).val();
  str += id + "/" + value + ", ";
});
```

### Radio 선택 처리

```javascript
var grpCd = sessionStorage.getItem("grpCd");
if (grpCd) {
  $("input:radio[name='grpCd'][value='" + grpCd + "']").prop("checked", true);

  $("#grpCd1").prop("checked", true);
}
```

### Radio Event

```javascript
$("input[type=radio][name=grpCd]").change(function () {
  var arr = $("input[type=radio][name=grpCd]");
  console.log($(this).val());
  for (var i = 0; i < arr.length; i++) {
    if ($(this).val() != $(arr[i]).val()) {
      $(arr[i]).siblings("input").val("");
    }
  }
});
```

### Radio disable 관련

```javascript
$("#rdPink").prop("disabled", true);

/* 전체 Disabled 설정 */
$('input[name="grpCd"]').each(function () {
  $(this).prop("disabled", true);
});
```

### Radio 개수 확인

```javascript
var num = $('input[name="grpCd"]').length;

var num = $('input[name="grpCd"]:checked').length;

var num = $('input[name="Radioame"]:not(:checked)').length;
```

## jQuery Checkbox

### Checkbox 선택여부 확인

```javascript
if (!$("input:checkbox[name=agree]").is(":checked")) {
  alert("Pleaze Check!!");
  return false;
}
```
