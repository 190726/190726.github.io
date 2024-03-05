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

## 언제 Enum 을 사용해야 할까?
 - 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거타입을 사용

## 기본 Enum 만드는 과정
```java
/*1. enum 변수 정의*/
/*2. private final String text 작성*/
/*3. 생성자와 get메서드*/
@Getter
@RequiredArgsConstructor
public enum ProductType {

    HANDMADE("제조 음료"),
    BOTTLE("병 음료"),
    BAKERY("베이커리");

    private final String text;

    public static ProductType findByTest(String findText) {
    	for(ProductType p:values()) {
    		if(p.text.equals(findText)) return p;
    	}
    	throw new IllegalStateException("no productType matching find Text");
    }

    public static boolean containsStockType(ProductType type) {
        return List.of(BOTTLE, BAKERY).contains(type);
    }
    
    public static List<ProductType> canSellType(){
    	return List.of(HANDMADE, BOTTLE);
    }
}
```

## 활용 Enum 상수별 메서드 구현
```java
@Getter
@RequiredArgsConstructor
public enum MathOperation {
	
	PLUS("+"){
		public double apply(double x, double y) {return x + y;}},
	MINUS("+"){
		public double apply(double x, double y) {return x-y;}},
	TIMES("+"){
		public double apply(double x, double y) {return x * y;}},
	DIVIDE("+"){
		public double apply(double x, double y) {return x / y;}
	};
	
	private final String symbol;

  private static final Map<String, MathOperation> stringToEnum = 
			Stream.of(values()).collect(Collectors.toMap(Object::toString, e->e));
	
	public static Optional<MathOperation> fromString(String symbol){
		return Optional.ofNullable(stringToEnum.get(symbol));
	}

	public abstract double apply(double x, double y);
}
```
## 활용 전략 열거 타입 패턴
```java
@Getter
@RequiredArgsConstructor
public enum PayrollDay {
	
	MONDAY   (PayType.WEEKDAY), 
	UESDAY   (PayType.WEEKDAY), 
	WEDNEDAY (PayType.WEEKDAY), 
	THURSDAY (PayType.WEEKDAY), 
	FRIDAY   (PayType.WEEKDAY), 
	SATURDAY (PayType.WEEKEND), 
	SUNDAY   (PayType.WEEKEND);
	
	private final PayType payType;
	int pay(int minutesWorked, int payRate) {
		return payType.pay(minutesWorked, payRate);
	}
	
	enum PayType{
		WEEKDAY{
			int overTimePay(int mins, int payRate) {
				return mins * payRate;
			}
		},
		WEEKEND{
			int overTimePay(int mins, int payRate) {
				return mins * payRate * 2;
			}
		};
		abstract int overTimePay(int mins, int payRate);

		int pay(int minutesWorked, int payRate) {
			int basePay = 10000;
			return basePay + overTimePay(minutesWorked, payRate);
		}
	}
}
```
```
