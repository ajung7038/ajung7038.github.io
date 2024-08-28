---
title: "[Java] Item 03 : private 생성자나 열거 타입으로 싱글턴임을 보증하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-08-28 10:14:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 03 : private 생성자나 열거 타입으로 싱글턴임을 보증하라


## 🫧 싱글턴 (singleton)

인스턴스를 오직 하나만 생성할 수 있는 클래스

전형적인 예로는 함수와 같은 무상태 (stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다.

### ✨ 생성 방식

#### 1. public static final 필드 방식의 싱글턴

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }

  public void leaveTheBuilding() { ... }
}
```

Elvis 클래스가 초기화될 때 딱 한 번 만들어진 인스턴스를 다시 만드는 방법은 없다. 따라, Elvis 인스턴스가 전체 시스템에서 하나뿐임이 보장된다. 싱글턴 클래스이다.

#### 2. 정적 팩터리 방식의 싱글턴

- 정적 팩터리 메서드를 public static 멤버로 제공

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }

  public void leaveTheBuilding() { ... }
}
```

객체를 얻고 싶을 때는 Elvis.getInstance()를 통해 얻어올 수 있다. 항상 같은 객체의 참조를 반환하는 것이 보장되어 있기 때문이다.

2번 방식의 장점을 열거하자면 다음과 같다.

1. API 변경 없이 싱글턴이 아니게 변경 가능
2. 정적 팩터리를 제네릭 싱글 팩터리로 변경 가능
3. 정적 팩터리 메서드 참조를 공급자로 사용

1, 2 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다.

1번 방식과 2번 방식으로 만든 클래스를 직렬화하려면 단순히 Serializable를 구현한다고 선언하는 것만으로는 부족하기 때문에 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야 한다.

자칫 잘못 했다가는 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어질 수 있다.

```java
private Object readResolve() {
  return INSTANCE;
}
```

마지막으로, 책에서 가장 추천하는 방법이 있다.

#### 3. 열거 타입 방식의 싱글턴

- 원소가 하나인 열거 타입 선언

```java
public enum Elvis {
  INSTANCE;
  
  public void leaveTheBuilding() { ... }
}
```

이는 public 필드 방식과 비슷하짖만, 더 간결하고, 추가 노력 없이 직렬화할 수 있으며 리플렉션 공격도 방어해 준다.

💡 권한이 있는 클라이언트는 리플렉션 API를 통해 private 생성자를 호출하는 공격을 할 수도 있다. 이러한 공격을 방어하기 위해 1, 2번 방식에는 추가적인 작업을 해 주어야 한다. 이는 아이템 65에서 더 자세하게 다룬다.

따라, 대부분 상황에서는 원소가 하나뿐인 열거 타입을 활용해 싱글턴을 만들되, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 다른 방법을 사용하도록 하자.


# 📌 추가 공부 및 질문

## 🫧 호출하는 스레드별로 다른 인스턴스를 어떻게 넘겨주는가?

## 🫧 공급자 (Supplier)