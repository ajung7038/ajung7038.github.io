---
title: "[Java] Item 13 : clone 재정의는 주의해서 진행하라
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-10-01 22:01:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 13 : clone 재정의는 주의해서 진행하라

## 🫧 Cloneable

Object.clone() 메서드는 인스턴스 객체를 복제하는 경우 사용된다.

이 메서드는 기본적으로 protected 접근 권한을 갖고 있기 때문에 사용하는 경우 public으로 재정의하여 사용해야 한다.

이때 데이터 보호를 이유로 Cloneable 인터페이스를 구현 해야 사용이 가능하다.

### ✨ 코드

```java
package java.lang;
public interface Cloneable {
}
```

놀랍게도 이게 끝이다.

대개 인터페이스를 구현한다는 것은 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위이다.

그러나 Cloneable는 메서드 하나 없이 오로지 껍데기만 들어 있다. 기능 제공보다는 상위 클래스에 정의된 protected 메서드의 "동작 방식 변경"에 초점을 맞췄기 때문이다.

Object.clone() 메서드는 기본적으로 protected 접근 권한을 갖고 있기 때문에, Cloneable 메서드를 통해 protected -> public으로 동작 방식을 변경하는 것이다.

### ✨ 동작 방식

Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException 예외를 던진다.

실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 그러나 이를 만족시키기 위해서는 허술한 프로토콜을 지켜야 한다.

실제로도 clone 메서드 일반 규약은 허술하다.

## 🫧 Clone 메서드 일반 규약

: 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어던 객체 x에 대해 다음 식은 참이다.

1) x.clone() != x

    -> 인스턴스가 다르므로 원본값과 clone된 인스턴스는 다른 객체이다.

2) x.clone().getClass() == x.getClass()

    -> clone한 객체의 클래스 타입과 원본 클래스 타입은 동일하다.

3) x.clone().equals(x)

    -> 논리적 동치를 뜻한다. x의 내용 (다른 인스턴스에 똑같은 값을 편의상 내용이라 지칭) 과 clone된 인스턴스의 내용이 같다.

    -> 이는 일반적으로 참이지만, 필수는 아니다.

4) x.clone().getClass() == x.getClass()


강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷하다.

이러한 clone 규약이 허술한 이유는 바로 다음 문제에 드러난다.

만약 clone 메서드 호출이 아닌 생성자를 호출해 인스턴스를 얻는다고 하더라도 컴파일러에서 문제를 잡아낼 수 없다. clone 규약을 어기지 않고 생성자를 통해 '복사'했기 때문이다.

그러나 이 클래스의 하위 클래스에서 super.clone을 호출하는 순간 문제가 발생한다. 상위 클래스는 clone 재정의가 아닌 생성자를 호출했기 때문이다. 결국 하위 클래스의 타입은 상위 클래스가 아닌 상위 클래스의 상위 클래스 타입을 반환하게 될 것이다. (만약 상위 클래스 타입의 상위 클래스 타입이 clone()을 재정의했다면)

## 🫧 Cloneable 구현

Cloneable 구현은 크게 두 가지로 나뉜다.

1) 기본 타입 or 불변 객체 참조
2) 가변 객체 참조

### ✨ 기본 타입 or 불변 객체 참조

```java
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertonError();
    }
}
```

- 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입으로 반환될 수 있도록 구현 해야 한다.

    -> 자바의 공변 반환 타이핑
    
#### 💡 (PhoneNumber) super.clone(); 처럼 형 변환을 자동으로 해 주자

### ✨ 가변 객체 참조

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (sie == 0)
            throw new EmpthStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

이러한 객체를 clone 메서드를 통해 복제할 수 있을까?

반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조한다. 원본이나 복제본이 수정되는 순간 다른 하나도 함께 수정되는 문제가 생기게 되는 것이다.

따라서, 스택 내부 정보를 복사 해야 한다.

```java
@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

## ✨ 해시테이블에서의 복제

전과 똑같이 clone을 재귀적으로 호출하는 경우 원본과 같은 연결 리스트를 참조하게 된다. 따라서 해시 테이블의 경우 각 버킷을 구성하는 연결 리스트를 복사해야 한다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry (Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            return new Entry(key, value, next == null? null : next.deepCopy());
        }
    }

    @Override public HashTable clone() {
        try {
            HashTable result = (HastTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i=0; i<buckets.length; i++) {
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

해당 버킷 배열만 clone하는 것이 아닌, 연결 리스트까지 복제를 해 줘야 한다.

그러나 재귀 호출로 인해 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있다. 따라서 재귀 호출 대신 반복자를 써서 순회하는 방향으로 코드를 짜야 한다.

```java
Entry deepCopy() {
    Entry result = new Entry (key, value, next);
    for (Entry p = result; p.next != null; p= p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

## 🫧 clone 재정의 시 주의사항

1) 재정의될 수 있는 메서드를 호출하지 마라.

    -> final or private으로 설정해 변경 가능성을 없애라

2) 사용의 편의성을 위해 재정의한 메서드에서는 검사 예외를 던지지 마라.
3) 상속용 클래스는 Cloneable을 구현하지 마라.
4) clone 메서드를 재정의하고 동기화해야 한다.


## 🫧 정리

Cloneable을 구현하는 모든 클래스는 clone을 재정의해라
    
    -> 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경

clone을 재정의할 시 super.clone 호출 후 가변 객체까지 복제될 수 있도록 구현해라.

또는 Cloneable을 사용하지 않는다면 복사 생성자와 복사 팩터리를 구현해라.

여기서 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

# 📌 참고자료
- [Cloneable 메서드](https://inpa.tistory.com/entry/JAVA-%E2%98%95-Object-%ED%81%B4%EB%9E%98%EC%8A%A4-clone-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%96%95%EC%9D%80-%EB%B3%B5%EC%82%AC-%EA%B9%8A%EC%9D%80-%EB%B3%B5%EC%82%AC)