---
title: "[Java] Item 34 : int 상수 대신 열거 타입을 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-01-25 12:25:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 34 : int 상수 대신 열거 타입을 사용하라

## 🫧 열거 타입

### ✨ 정수 열거 패턴

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.

ex) 사게절, 태양계의 행서, 카드 게임의 카드 종류

자바에서 열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해 사용했다.

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_FPIPPIN = 1;
```

그러나 이런 식의 `정수 열거 패턴`을 사용하면 다음과 같은 단점이 뒤따른다.

1. 타입 안전을 보장할 수 없다.
2. 표현력이 좋지 않다.
3. 별도 이름공간을 지정하지 않기 때문에 접두어를 써서 이름 충돌을 방지해야 한다.
4. 컴파일 이후 값 변경이 어려워 상수의 값이 변경될 경우 다시 컴파일 해야 한다.
5. 문자열로 출력하기 까다롭다.

특히, 그 값을 출력하거나 디버거로 봤을 때 의미가 아닌 단순 숫자로만 보여 그것이 무엇을 의미하는지 파악하기 어렵다.

### ✨ 문자열 열거 패턴 (string enum pattern)

따라 정수 대신 문자열 상수를 사용하는 변형 패턴인 `문자열 열거 패턴 (string enum pattern)`도 나오게 되었지만 좋은 방식은 아니다.

상수의 의미를 출력하는 점에서는 더 좋아 보일 수 있으나, 하드코딩이 필요하다는 점에서 어쩌면 정수 열거 패턴보다 더 나쁘다고 할 수 있다.

문자열에 오타가 있어도 컴파일러가 확인할 수 없고, 문자열 비교에 따른 성능 저하가 발생하기 때문이다.

### ✨ 열거 타입 (enum type)

이러한 정수 열거 패턴과 문자열 열거 패턴의 단점을 보완하기 위해 자바에서는 `열거 타입 (enum type)`을 제시했다.

다음은 열거 타입의 가장 단순한 형태이다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

### ✨ 열거 타입의 특징

1. 열거 타입 자체는 클래스이다.
2. 인스턴스 통제된다.
<br/>-> 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재

3. 컴파일 타입 안전성을 제공한다.
4. 정수 열거 패턴과 달리 각자의 이름 공간이 있어 이름이 같은 상수도 존재할 수 있다.
5. 순서에 상관 없어 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일할 필요가 없다.
6. toString 메서드를 통해 문자열을 적합하게 출력할 수 있다.

## 🫧 열거 타입 & 메서드와 필드

### ✨ 필드

뿐만 아니라, 열거 타입은 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스 또한 구현 가능하다.

그렇다면 어떨 때 이러한 메서드, 필드, 인터페이스를 사용하는 것일까?

예를 들면 태양계의 여덟 행성에 관해 각각의 계산을 행해야 한다고 가정하자.

각 행성의 질량과 반지름에 따라 표면중력이 달라지고, 어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을 때의 무게 또한 계산이 가능하다.

따라, 열거 타입으로 질량과 반지름을 받아 각 행성에 따른 표면중력을 계산한다고 할 때 이러한 메서드, 필드가 필요한 것이다.

### ✨ 메서드

상수가 서로 다른 데이터와 연결되어 있는 것에서 더 나아가, 상수마다 동작이 달라져야 하는 상황을 살펴보자.

예를 들어 사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행했으면 한다고 해 보자.

### ✨ 메서드 예시 - 사칙연산 계산기

1. switch문을 이용한 상수의 값에 따라 분기하는 방식

```java
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  public double apply(double x, double y) {
    switch(this) {
      case PLUS: return x+y;
      case MINUS: return x-y;
      case TIMES: return x*y;
      case DIVIDE: return x/y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```

이러한 코드는 깨지기 쉽다.

만약 새로운 상수를 추가한다면 해당 case문도 추가해야 하며, 혹시 깜빡한다면 런타임 오류가 날 것이다.

이러한 방식의 단점을 극복하기 위해 상수별로 다르게 동작하는 코드를 구현할 수 있도록 "추상 메서드" 를 제공한다.

이러한 방식을 `상수별 메서드 구현(constant-specific method implementation)`이라 한다.

```java
public enum Operation {
  PLUS { public double apply (double x, double y) { return x+y;}},
  MINUS { public double apply (double x, double y) { return x-y;}},
  TIMES { public double apply (double x, double y) { return x*y;}},
  DIVIDE { public double apply (double x, double y) { return x/y;}},

  public abstract double apply(double x, double y);
}
```

apply를 반드시 정의해야 하기 때문에 오류를 잡아내기도 쉽다.

## 🫧 fromString 메서드

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.

열거 타입의 toString 메서드를 재정의할 때 to String이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공된다.