---
title: "[Java] Item 35 : ordinal 메서드 대신 인스턴스 필드를 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-01-25 12:45:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 35 : ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다.

그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

```java
public enum Ensemble {
  SOLO, DUET, TRIO ... DECTET;

  public int numberOfMusicians() {return ordinal() + 1};
}
```

위 코드는 ordinal을 사용해 위치를 반환하는 코드이다.

보기에는 문제 없어 보이나, 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.

또한, 값을 중간에 비워둘 수 없다.

이렇듯 ordinal 함수를 쓰면 코드가 깔끔하지 못할 뿐더러 쓰이지 않는 값이 많아질수록 실용성이 떨어진다.

따라, 이러한 해결책은 다음과 같다.

> 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.

```java
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3) ... DECTET(10);

  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```

따라서, ordinal은 EnumSet, EnumMap과 같이 열거 타입 기반의 범용 자료구조에 쓰지 않는 한 절대 사용하지 말고 인스턴스 필드를 사용하라는 것이다.