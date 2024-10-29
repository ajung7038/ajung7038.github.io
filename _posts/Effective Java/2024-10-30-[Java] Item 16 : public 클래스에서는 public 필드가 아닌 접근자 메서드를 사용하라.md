---
title: "[Java] Item 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-10-30 08:20:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

인스턴스의 필드를 모아두는 클래스가 하나 있다고 가정하자.

```java
class Point {
    public double x;
    public double y;
}
```

해당 클래스의 문제는 데이터 필드에 직접 접근할 수 있다는 점이다. 객체 지향의 특징 중 하나인 캡슐화의 이점을 제공하지 못하는 것이다.

따라서, 이러한 것들은 다음 코드와 같이 변경되어 왔다. 필드들의 접근을 private로 모두 바꾸고 public 접근자 (getter) 을 추가하면 이러한 문제는 쉽게 해결된다.

```java
class Poing {
    private double x;
    private double y;

    public Point (double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX (double x) { this.x = x; }
    public void setY (double y) { this.y = y; }
}
```

만약 이 클래스가 public 클래스인 경우 올바른 해결 방법이 될 것이다.

그러나 package-private 클래스 또는 private 중첩 클래스인 경우 이는 불필요한 코드가 된다. 데이터 필드를 노출해도 그 클래스가 표현하려는 추상 개념만 올바르게 표현한다면 문제가 될 것이 없기 때문이다.

package-private 클래스나 private 중첩 클래스는 필드를 노출하는 편이 나을 때도 있다. 외부에서 접근이 되지 않기 때문에 문제가 될 것은 없다.

