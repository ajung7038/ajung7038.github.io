---
title: "[Java] Item 24 : 멤버 클래스는 되도록 static으로 만들라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-12-17 07:44:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 24 : 멤버 클래스는 되도록 static으로 만들라

## 🫧 중첩 클래스
: 다른 클래스 안에 정의된 클래스

자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

### ✨ 중첩 클래스 종류
1. 정적 멤버 클래스
2. (비정적) 멤버 클래스
3. 익명 클래스
4. 지역 클래스

정적 멤버 클래스를 제외한 나머지 세 개의 중첩 클래스는 모두 내부 클래스이다.

## 🫧 정적 멤버 클래스

- 다른 클래스 안에 선언
- 바깥 클래스의 private 멤버에 접근 가능

```java
public class Animal {
    private String name = "cat";

    // 열거 타입도 암시적 static
    public enum Kinds {
        MAMMALS, BIRDS, FISH, REPTILES, INSECT
    }

    private static class PrivateSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();

            // 바깥 클래스인 Animal의 private 멤버 접근
            System.out.println("private" + outerClass.name);
        }
    }

    public static class PublicSample {
        private int temp;

        public void method() {
            Animal outerClass = new Animal();

            // 바깥 클래스인 Animal private 멤버 접근
            System.out.println("public" + outerClass.name);
        }
    }
}
```

Animal 클래스 내 PrivateSample, PublicSample 클래스가 선언된 상황이다. 이 코드에서 다음과 같이 도출이 가능하다.

- 정적 멤버 클래스 PrivateSample와 PublicSample 클래스
- PrivateSample과 PublicSample 모두 Animal의 private 필드인 name을 포함한 모든 멤버 및 메서드에 접근 가능
- 다른 정적 멤버와 똑같은 접근 규칙을 가짐
<br /> -> ex) private으로 선언하면 바깥 클래스에서만 접근 가능. 예시에서는 PrivateSample의 바깥 클래스인 Animal에서만 PrivateSample에 접근 가능하다.

```java
public class Main {
    public static void main(String[] args) {
        // 아래 코드는 오류: PrivateSample은 private이므로 외부 클래스에서 접근 불가
        Animal.PrivateSample ps = new Animal.PrivateSample();  // 컴파일 오류
    }
}
```

## 🫧 비정적 멤버 클래스

정적 멤버 클래스에 static이 붙지 않은 것

- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.

여기서 정규화된 this란 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법을 뜻한다.

### ✨ 비정적 멤버 클래스의 사용
- 어댑터 : 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
- Map 인터페이스 구현체 : 컬렉션 뷰 구현 시 비정적 멤버 클래스 사용

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여 정적 멤버 클래스로 만들자!

그렇지 않다면 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 발생하게 된다.

## 🫧 익명 클래스

- 쓰이는 시점에 선언과 동시에 인스턴스가 생성
- 비정적인 문백에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 상수 변수 이외의 정적 멤버는 가질 수 없다. (상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.)

최근 자바 람다가 익명 클래스를 대체하는 추세로, 자바 람다 지원 전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 사용하거나 정적 팩터리 메서드 구현 시 사용하였다.

## 🫧 지역 변수 클래스
- 지역 변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언 가능하다.
- 유효범위 또한 지역변수와 같다.

### ✨ 지역 변수 클래스 특징
- 이름이 있고, 재사용이 가능하다.
- 비정적 문맥에서 사용될 때만 바깥 인스턴스 참조가 가능하다.
- 정적 멤버를 가질 수 없다.


## 🫧 참고 자료
[[Java] 중첩 클래스의 종류 (feat. 멤버 클래스는 static으로!)_티스토리 | 연로그](https://yeonyeon.tistory.com/205)