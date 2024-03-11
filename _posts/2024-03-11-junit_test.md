---
title: "[java] junit and mockito"
excerpt: "자바 테스트 기법"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/jnit-mockito/

toc: true
toc_sticky: true

date: 2024-03-11
last_modified_at: 2024-03-11
---

## maven pom.xml 설정
```xml
        <!--mockito 코어-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>3.12.4</version>
            <scope>test</scope>
        </dependency>
        <!--junit5-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
        <!--Mockito Extension(모키토 초기화 작업등 자동화)-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>4.0.0</version>
            <scope>test</scope>
        </dependency>
        <!-- assertj (fluent api)-->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.25.1</version>
            <scope>test</scope>
        </dependency>
```
