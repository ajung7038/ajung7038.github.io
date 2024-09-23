---
title: "[Java] Item 10 : equals는 일반 규약을 지켜 재정의하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-09-23 12:47:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 10 : equals는 일반 규약을 지켜 재정의하라

## 🫧 Object 클래스
- 모든 클래스의 최상위 클래스
- 객체를 만들 수 있는 구체 클래스

-> 모든 클래스는 object에서 상속 받고, Object 클래스의 메서드 중 일부는 재정의해서 사용할 수 있다. (재정의를 염두에 두고 설계)

## 🫧 equals 메서드 재정의

### ✨ equals 메서드 재정의는 왜 필요할까?

equals는 "논리적 동치성" 을 비교하기 위해 사용된다.

만약 재정의되지 않는다면, 가장 상위 객체인 Object.equals의 메서드를 사용하게 되고, 이는 "=="과 같은 역할을 하기 때문에 객체와 같은 참조 타입에서는 값이 아닌 주소가 같아야 이를 같다고 인정한다.

따라서, 참조 타입에서도 값이 같은 경우 같다고 판단하고 싶다면 equals 메서드를 재정의하거나 이미 재정의된 메서드를 사용해야 한다.

#### 💡 사용 예

java.util.regex.Pattern은 equals를 재정의해 사용한 예시이다.

두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지 검사할 때 사용한다.


### ✨ 언제 equals 메서드를 재정의 하는가?
논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되어 있지 않을 떄
<br/> -> 주로 '값' 클래스가 이 유형에 해당
<br /> -> 논리적 동치성을 확인하도록 재정의한다면 값을 비교할 수도 있고, Map 키와 Set의 원소로도 사용이 가능함.


### ✨ equals 메서드 재정의를 하지 말아야 할 때
1) 각 인스턴스가 본질적으로 고유할 때
<br/> -> 값이 아닌 동작하는 개체를 표현하는 경우
<br/> -> ex, Thread

2) 인스턴스의 '논리적 동치성'을 검사할 필요가 없을 떄
3) 상위 클래스에서 재정의한 equals가 하위 클래스에서도 딱 들어맞을 때
4) 클래스가 private이거나 package-private이고, equals 메서드를 호출할 일이 없을 때

### ✨ equals 메서드 재정의 시 반드시 따라야 하는 일반 규약 (Object 명세)

* equals는 동치 관계를 구현하며, 다음을 만족한다.

1. 반사성
<br/> -> x.equals(x) == true

2. 대칭성
<br/> -> x.equals(y) == y.equals(x)

3. 추이성
<br/> -> x.equals(y) == y.equals(z) == z.equals(x)

4. 일관성

5. null-dksla
<br/> -> x.equals(null) == false



#### 💡 여기서 말하는 동치 관계란, 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산을 말한다.
- 나눠진 부분집합을 동치류 (동치 클래스) 로 지칭하며, 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환할 수 있어야 한다.


## 🫧 동치 관계를 만족시키기 위한 다섯 가지 요인

### ✨ 1. 반사성 : 객체는 자기 자신과 같아야 함
### ✨ 2. 대칭성 : 서로에 대한 동치 여부에 똑같이 답해야 함.
<br/> -> ex, 대소문자를 구분하지 않는 문자열 클래스

### ✨ 3. 추이성
<br /> -> 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 경우 "추이성"이 깨지기 쉬우므로 조심 해야 함.

### ✨ 4. 일관성
### ✨ 5. null-아님
<br/> 명시적으로 null 검사보다는 !를 활용한 묵시적 null 검사가 더 낫다.

💡 그렇다면 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법이 없을까?

instanceof 대신 getClass로 바꾸면 이 문제를 해결할 수 있어 보이지만, 사실은 아니다.

아래 코드는 리스코프 치환 원칙에 위배되는 예이다.

[liskov87] 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.
(point 클래스의 하위 클래스 또한 Point 클ㄹ스로써 활용될 수 있어야 한다.)

```java
@Override
public boolean equals (Object o) {
  if (o == null || o.getClass() != getClass())
    return false
  Point p = (Point) o;
  return p.x == x & p.y == y;
}
```

해답은 바로 "상속 대신 컴포지션을 사용하라"는 아이템 18의 조언을 따르면 된다. 정확한 방법은 아니지만, 우회해서 이를 사용할 수 있다.

```java
public class ColorPoint {
  private final Point point;
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }

  /**
   * 이 ColorPoint의 뷰 반환
   */
  public Point asPoint() {
    return point;
  }

  @Override public boolean equals (Object o) {
    if (!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

## 🫧 equals 사용과 주의사항

### ✨ 양질의 equals 메서드 구현법
1) "==" 연산자를 사용해 입력이 자기자신의 참조인지 확인
2) instanceof 연산자로 입력이 올바른 타입인지 확인
3) 입력을 올바른 타입으로 형변환
4) 입력 객체와 자기자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사

#### 💡 float와 doubl을 제외한 기본 타입 필드는 == 연산자로, 참조 타입 필드는 각각의 equals 메서드로, float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.

참고로, Float.equals, Double.equals 메서드를 대신 사용할 수 있지만, 이 메서드들은 오토박싱을 수반할 수 있으므로 성능 상 좋지 않다.

### ✨ equals 메서드 구현 예

```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;

  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = rangeCheck(areaCode, 999, "지역코드");
    this.prefix = rangeCheck(prefix, 999, "프리픽스");
    this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
  }

  private static short rangeCheck(int val, int max, String arg) {
    if (val < 0 || val ? max)
      throw new IllegalArgumentException(arg + ": " + val);
    return (short) val;
  }

  @Override public boolean equals (Object o) {
    if (o == this)
      return true;
    if (!(o instanceof PhoneNumber)) 
      return false;
    PhoneNumber pn = (PhoneNumber) o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
  }
}
```

### ✨ 주의사항
1) equals를 재정의할 때는 hashCode도 반드시 재정의하자.
2) 너무 복잡하게 해결하려 들지 말자 (별칭 비교 X)
3) object 이외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.