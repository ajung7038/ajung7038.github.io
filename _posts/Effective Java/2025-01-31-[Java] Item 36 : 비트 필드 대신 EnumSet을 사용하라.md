---
title: "[Java] Item 36 : 비트 필드 대신 EnumSet을 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-01-31 11:05:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 36 : 비트 필드 대신 EnumSet을 사용하라

## 🫧 비트 필드 (정수 열거 패턴)

열거한 값들이 주로 집합으로 사용될 경우, 상수에 서로 다른 2의 거듭제곱을 할당한 `정수 열거 패턴`을 사용했다.

```java
public class Text {
  public static final int STYLE_BOLD = 1 << ; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKEHROUGH = 1 << 3; // 8

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값
  public void applyStyles(int styles) { ... }
}
```

다음과 같은 식으로 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 <strong>비트 필드(bit field)</strong>라고 부른다.

비트 필드는 다음과 같이 쓸 수 있다.

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

### ✨ 비트 필드를 왜 사용하는가?

비트 필드는 비트별 연산을 사용해 합집합, 교집합 등 집합 연산을 효율적으로 수행할 수 있다.

단순한 값이 아닌 하나의 세트로써 처리할 수도 있다.

그렇다면 여기서 질문을 하나 던질 수 있을 것이다.

왜 단순한 정수가 아닌 비트로써 이러한 값에 접근하는 것일까? 비트 필드를 사용하지 않고 정수로 단순히 세트를 처리할 수는 없을까?

그 해답은 "메모리 절약"과 관련이 있다.

비트 필드는 참-거짓 값을 여럿 결합해서 하나의 바이트로 만들어, 메모리를 절약한다.

Java에서 int 형은 4byte가 필요하다. 반면 비트 저장에는 1~2bit만 필요하기 때문에 메모리 자원 측에서 훨씬 효율적이다.

### ✨ 비트 필드의 장점

앞서 말한 비트 필드는 어쨌거나 정수 열거 패턴이다. 그러므로 당연히 정수 열거 상수의 단점 또한 가지고 있을 수밖에 없다.

뿐만 아니라, 다음과 같은 단점 또한 안고 있다.

1. 비트 필드의 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다.
2. 비트 필드 내 모든 원소를 순회하기 어렵다.
3. 최대 몇 비트가 필요한지 API 작성 시 예측해 타입을 선택해야 한다. (보통 int or long)

따라, 정수 열거 패턴의 단점과 여러 추가적인 단점을 안고 있는 비트 필드의 사용을 지양해야 한다.

## 🫧 EnumSet

이러한 단점을 보완하기 위해 나온 것이 EnumSet 클래스이다.

EnmSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 준다.

Set 인터페이스의 구현체 중 하나이며, 타입 안전할 뿐더러 다른 Set 구현체와 어울릴 수 있다.

그렇다면 EnumSet의 성능은 비트 필드보다 떨어지지 않을까?

아무래도 메모리 절약과 연관 있는 비트 필드의 단점을 보완하고, EnumSet이라는 클래스를 만들어 관리하므로 그렇게 질문할 수 있을 것이다.

이 문제에 관해 논의하려면 EnumSet의 구현을 살펴 보아야 한다.

EnumSet의 구현은 아래에 자세하게 설명할 것이다.

### ✨ EnumSet 구현

EnumSet은 <strong>비트 벡터</strong>로 구현되어 있다.

> 💡 여기서 비트 벡터란 중복되지 않는 정수 집합을 비트로 나타내는 방식을 말한다.

정수 집합 { 1, 2, 6 } 을 비트 벡터로 표현하면 다음과 같다.

```
정수	0	1	2	3	4	5	6	7	…

값의 유무	 	유	유	 	 	 	유	 	 

비트 벡터	0	1	1	0	0	0	1	0	0
```

이렇듯 비트 벡터를 사용할 때 원소가 총 64개 이하라면, 대부분의 경우 EnumSet을 long 변수 하나로 표현 가능하기에 비트 필드에 비견되는 성능을 보여준다.

원소가 64개 이하여야 하는 이유가 붙은 까닭은 하나의 값을 저장하는 데 한 비트가 필요하기 때문이다. (long 타입은 64bit)

특히, removeAll과 retainAll 같은 대량 작업 (비트 필드를 사용할 때 쓰는 것과 같은)은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

비트를 직접 다룰 때 흔히 겪는 오류 또한 EnumSet이 알아서 처리해 준다.

비트 필드의 단점을 상쇄하고 장점만 취하게 된 것이다.

### ✨ EnumSet 예시

앞서 보았던 비트 필드로 구현된 코드를 다시 수정해 보았다.

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

  // 어떠한 종류의 Set을 넘겨도 된다. 다만 EnumSet이 가장 좋다.
  public void applyStyles (Set<Style> styles) { ... }
}
```

왜 EnumSet<>이 아닌 Set<>을 선택했을까?

그것이 프로그램의 유연성을 높이기 때문이다.

모든 클라이언트가 EnumSet을 건네겠지만, 그럼에도 구현체가 아닌 인터페이스로 받는 것이 코드를 유연하게 만드는 지름길이다.

이는 아이템 64에서 더 자세하게 다룰 것이다.

해당 코드를 사용하는 클라이언트 코드는 아래와 같다.

```java
text.applyStyles(EnumSet.of(Sylte.BOLD, Style.ITALIC));
```

이렇듯 비트 필드를 사용하기보다는 비트 필드의 단점을 보완하고 장점만 취한 EnumSet을 사용하는 것이 타입 안전하고 좋은 코드를 짤 수 있는 방법이다.


## 🫧 참고 자료
[[JAVA] 3. 비트 필드의 개념과 사용, 문제점 | Velog | glenn_syj.log](https://velog.io/@glenn_syj/JAVA-3.-%EB%B9%84%ED%8A%B8-%ED%95%84%EB%93%9C%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%82%AC%EC%9A%A9-%EB%AC%B8%EC%A0%9C%EC%A0%90)