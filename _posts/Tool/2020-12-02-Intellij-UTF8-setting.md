---
title: 인텔리제이 한글 설정, 인텔리제이 UTF-8 설정
date: 2020-12-02-11:00
categories:
- Tool

tags:
- IDE
- Intellij

photos: 
- /post_images/intellij_UTF8_0.jpg

---

## 인텔리제이에서 한글이 잘 출력되게 해봅니다.
> 포맷을 UTF-8로 변경해 줍니다.

---

## VMOption 변경

<br>

![image](/post_images/intellij_UTF8_0.jpg)

* help > Edit Custom VM Options

<br>

![image](/post_images/intellij_UTF8_1.jpg)

* 15번쨰 라인처럼 `Dfile.encoding=UTF-8` 추가

---

## Editor 설정

<br>

![image](/post_images/intellij_UTF8_2.jpg)

* Setting > Editor

<br>

![image](/post_images/intellij_UTF8_3.jpg)

* Editor > File Encodings
    * 사진과 같이 Encoding 부분을 전부 `UTF-8` 로 변경
