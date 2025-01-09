---
title: "[Java] Item 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-01-09 15:59:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라

아이템 28에서 언급했다시피 매개변수화 타입은 불공변이다.

그러나 때로는 불공변 방식보다 유연한 무언가가 필요하다.

예를 들면 다음과 같다.

`Stack<Number>`로 선언한 스택이 있다고 가정하자. 여기에 Integer 타입의 원소를 넣으려고 시도하면 어떻게 될까?

Integer은 Number의 하위 타입이니 논리적으로는 잘 동작할 것 같지만, 그렇지 않다.

매개변수화 타입이 불공변이기 때문에 해당 원소는 들어갈 수 없다.

이를 위해 등장한 것이 바로 <strong>한정적 와일드카드</strong>이다.

## 🫧 한정적 와일드카드

한정적 와일드카드 타입이라는 특별한 매개변수화 타입에는 두 가지가 존재한다.

1. extends
2. super

해당 내용은 다음과 같이 쓰인다.

```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e);
}
```

이 extends는 방금 전 보았던 문제를 해결하기 위한 방법이다.

E의 하위 타입의 Iterable이 pushAll 메서드의 입력 매개변수 타입이라는 것이다.

마찬가지로, super 또한 비슷하게 쓰일 수 있으며 상위 타입과의 호환을 위해 사용된다.

## 🫧 PECS (펙스)

언제 어떤 와일드카드 타입을 써야 하는가?

해당 질문에 대한 대답은 PECS 원칙을 기억하면 쉽게 해결될 수 있다.

매개변수화 타입 T가 생산자라면 <? extends T> 를 사용하고, 소비자라면 <? super T>를 사용하면 된다.

Stack 예시에서 pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 Iterable<? extends E>를 사용하면 된다.

