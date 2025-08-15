---
title: "[Java] Item 89 : 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-08-15 17:27:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 89 : 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

싱글턴을 사용하면 인스턴스가 오직 하나만 만들어짐을 보장할 수 있다.

그러나 클래스의 선언에 `implements Serializable`을 추가하는 순간, 보이지 않는 생성자를 만들어낼 수 있어 싱글턴 보장이 깨지게 된다.

이를 해결하기 위해 다음과 같은 방식을 채택할 수 있다.

## 🫧 방법 1 : readResolve() 활용하기

다음과 같은 함수를 정의한다면, 싱글턴이 보장되리라 예상할 수 있다.

```java
private Object readResolve() {
  return INSTANCE;
}
```

역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다.

대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.

## 🫧 readResolve() 사용 시 주의사항

다만, readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.

그렇지 않으면 아이템 88에서 살펴본 MutablePeriod 공격과 비슷한 방식으로 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.

## 🫧 readResolve() 메서드가 수행되기 전 역직렬화된 객체의 참조를 공격하는 법

기본 아이디어는 다음과 같다.

> 싱글턴이 transient가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화되는데, 이때 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

다음 코드를 살펴보자.


### ✨ transient가 아닌 참조 필드를 가지고 있는 싱글턴 클래스

```java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
```


### ✨ 도둑 클래스

```java
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;

  private Object readResole() {
    // resolve되기 전의 Elvis 인스턴스의 참조 저장
    imersonator = payload;

    // favoriteSongs 필드에 맞는 타입의 객체를 반환
    return new String[] { "A Fool Such as I" };
  }

  private static final long serialVersionUID = 0;
}
```

### ✨ 실행 코드
```java
// ... (중략)
public static void main(String[] args) {
  // ElvisStealer.impersonator를 초기화한 다음,
  // 진짜 Elvis(Elvis.INSTANCE)를 반환한다.
  Elvis elvis = (Elvis) deserialize(serializedForm);
  Elvis impersonator = ElvisStealer.impersonator;

  elvis.printFavorites();
  impersonator.printFavorites();
}
```

이 프로그램을 실행하면 다음과 같은 결과가 출력될 것이다.

[Hound Dog, Heartbreak Hotel]
<br/>[A Fool Such as I]

왜 서로 다른 두 개의 Elvis 인스턴스가 생성되었을까?

### ✨ 공격 방법

readResolve 메서드와 인스턴스 필드 하나를 포함한 `도둑` 클래스가 있다.

이 인스턴스는 도둑이 `숨길` 직렬화된 싱글턴을 참조하는 역할을 한다.

직렬화된 스트림에서 싱글턴의 비휘발성 필드 (여기서는 favoriteSongs)를 이 도둑의 인스턴스로 교체한다.

이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환 고리가 만들어졌다.

싱글턴이 도둑을 포함하므로 싱글턴이 역직렬화될 때 도둑의 readResolve 메서드가 먼저 호출된다.

그 결과, 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인 (그리고 readResolve가 수행되기 전인) 싱글턴의 참조가 담겨 있게 된다.

## 🫧 방법 2 : 열거 타입 활용하기

기존 코드에서 문제가 되었던 favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만 Elvis를 원소 하나짜리 열거 타입으로 바꿔 이 문제를 해결할 수도 있다.

직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.

```java
public enum Elvis {
  INSTANCE;

  private String[] favoirteSongs = { "Hound Dog", "Heartbreak Hotel" };

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```