---
title: Python DataFrame 기본/생성/수정/삭제
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Python
  - DataFrame
toc: true
toc_sticky: true
toc_label: 목차
description: Python DataFrame 기본/생성/수정/삭제
article_tag1: Python
article_tag2: DataFrame
article_tag3:
article_section: Python DataFrame 기본/생성/수정/삭제
meta_keywords: Python, DataFrame
last_modified_at: 2022-07-09T00:00:00+08:00
---

[참고]({{ "https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html" }}){:target="\_blank"} 사이트를 통해 재정리.
https://3months.tistory.com/292
## DataFrame 기본 형태

```python
import pandas as pd
df = pd.DataFrame(data=None, index=None, columns=None, dtype=None, copy=False)
```

* data - DataFrame을 생성할 데이터
* index - 각 Row에 대해 Label을 추가 ( 옵션 )
* columns - 각 Column에 대해 Label을 추가 ( 옵션 )
* dtype - 각 Column의 데이터 타입 명시 ( 옵션 )

> DataFrame.from_records | 튜플의 생성자이며 배열도 기록합니다.
> 
> DataFrame.from_dict | 튜Series의 dicts, arrays 또는 dicts에서.
> 
> read_csv | 튜쉼표로 구분된 값(csv) 파일을 DataFrame으로 읽어옵니다.
> 
> read_table | 튜일반 구분 파일을 DataFrame으로 읽어옵니다.
> 
> read_clipboard | 튜클립보드에서 DataFrame으로 텍스트를 읽습니다.

## DataFrame 생성

```python
import numpy as np
from pandas import Series, DataFrame

# dictionary를 사용한 DataFrame 생성
raw_data = {'col0': [1, 2, 3, 4],
            'col1': [10, 20, 30, 40],
            'col2': [100, 200, 300, 400],
            'col3': ['kim', 'lee', 'ahn', 'james']}

data = DataFrame(raw_data)
print(data)
print(data.dtypes)
print(data['col0'])
print(data['col0'].to_list())

   col0  col1  col2   col3
0     1    10   100    kim
1     2    20   200    lee
2     3    30   300    ahn
3     4    40   400  james
col0     int64
col1     int64
col2     int64
col3    object

0    1
1    2
2    3
3    4
Name: col0, dtype: int64

[1, 2, 3, 4]

#  List를 사용한 DataFrame 생성
raw_data = [['Choi',22],['Kim',48],['Joo',32]]

data = DataFrame(raw_data)
print(data)
      0   1
0  Choi  22
1   Kim  48
2   Joo  32

# index와 columns을 사용한 DataFrame 생성
raw_data = [{'a':1, 'b':2},{'a':5, 'b':10, 'c':20}]

data = DataFrame(data, index = ['first','second'], columns=['a','b']) 

print(data)
        a   b
first   1   2
second  5  10

# columns 순서대로 생성 가능
raw_data = {'open':  [11650, 11100, 11200, 11100, 11000],
            'high':  [12100, 11800, 11200, 11100, 11150],
            'low' :  [11600, 11050, 10900, 10950, 10900],
            'close': [11900, 11600, 11000, 11100, 11050]}
data = DataFrame(raw_data, columns=['open', 'high', 'low', 'close'])
print(data)
    open   high    low  close
0  11650  12100  11600  11900
1  11100  11800  11050  11600
2  11200  11200  10900  11000
3  11100  11100  10950  11100
4  11000  11150  10900  11050

# numpy를 사용한 DataFrame 생성
raw_data = np.array([[1,2,3],[4,5,6]])

data = DataFrame(raw_data, columns=['col1','col2','col3'])
print(data)
   col1  col2  col3
0     1     2     3
1     4     5     6

```

## DataFrame 수정



## DataFrame 삭제

