---
title: "[java] 자바 기본 문법 모음"
excerpt: "자바 문법 플래시 카드"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/java-flash/

toc: true
toc_sticky: true

date: 2024-02-20
last_modified_at: 2024-02-20
---

## 쉬프트 연산 <<와 >>의 차이
```java
int a = 2;
/*왼쪽으로 1비트 이동시 값이 2배 증가*/
a << 1 //4
a << 2 //8
a << 3 //16
int b = 8;
b >> 1 //4
b >> 2 //2
b >> 3 //1
b >> 4 //0
b >> 5 //0
```
## 문자열을 구분자로 구분해서 결합
```java
String.join(",","a","b","c") //a,b,c
```
## Long, long 을 int로 변환
```java
Long a1 = 4L;
long a2 = 4L;
int b1 = a1.intValue();
int b2 = Long.valueOf(a2).intValue();
```
