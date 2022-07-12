---
title: Python tkinter Simple Start Exsample
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Python
  - tkinter
toc: true
toc_sticky: true
toc_label: 목차
description: Python tkinter Simple Start Exsample
article_tag1: Python
article_tag2: tkinter
article_tag3:
article_section: Python tkinter Simple Start Exsample
meta_keywords: Python, tkinter
last_modified_at: 2022-07-12T00:00:00+08:00
---

[참고]({{ "https://076923.github.io/posts/Python-tkinter-5/" }}){:target="\_blank"} 사이트를 통해 재정리.

## Python tkinter Simple Start Exsample

```python
import os
import tkinter
import pandas as pd 
from tkinter import filedialog
from tkinter import messagebox

window = tkinter.Tk()

window.title("File Converter")
window.geometry("640x400+100+100")
window.resizable(True, True)

label=tkinter.Label(window, text=", csv파일을 선택하면 자동으로 변환 됩니다.")
label.pack()

# 파일변환
def convertFile():
    list_file = [] #파일 목록 담을 리스트 생성
    files = filedialog.askopenfilenames(initialdir="/", title = "파일을 선택 해 주세요", filetypes = (("*.xlsx","*xlsx"),("*.xls","*xls"),("*.csv","*csv")))
    #files 변수에 선택 파일 경로 넣기
    if files == '':
        # messagebox.showwarning("경고", "파일을 추가 하세요")    #파일 선택 안했을 때 메세지 출력
        label.config(text="파일을 선택하지 않았습니다.")
    else:
        messagebox.showwarning("확인", "파일 변환이 완료 되었습니다.")    #파일 선택 안했을 때 메세지 출력
        label.config(text="파일 변환이 완료 되었습니다.")
    print(files) #files 리스트 값 출력

button = tkinter.Button(window, text="파일선택", overrelief="solid", width=15, command=convertFile, repeatdelay=100, repeatinterval=100)
button.pack()

label = tkinter.Label(window, text="")
label.pack()

listbox = tkinter.Listbox(window, selectmode='extended', height=0)
listbox.insert(0, "0번")
listbox.insert(1, "1번")
listbox.insert(2, "2번")
listbox.insert(3, "3번")
listbox.insert(4, "4번")
listbox.delete(1, 2)
listbox.pack()

window.mainloop()
```
