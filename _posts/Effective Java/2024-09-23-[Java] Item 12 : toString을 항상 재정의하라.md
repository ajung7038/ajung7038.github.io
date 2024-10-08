---
title: "[Java] Item 12 : toString을 항상 재정의하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-09-23 16:38:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 12 : toString을 항상 재정의하라

## 🫧 Object에서의 toString

어떤 객체를 반환하려 할 때, print를 찍어보면 `PhoneNumber@adbbd처럼 단순히 클래스_이름@16진수_해시코드` 만을 반환한다.

그러나 우리가 필요한 정보는 707-867-5309와 같은 어떠한 "정보"를 담고 있는 객체의 내용일 것이다.

또한 toString의 규약은 "모든 하위 클래스에서 이 메서드를 재정의하라" 고 한다.

따라서, Object의 toString을 재정의 해 필요한 정보를 얻어내는 것이 나을 것이다.

## 🫧 재정의된 toString

실전에서 toString은 그 객체가 가진 주요 정보를 모두 반환하는 게 좋다.

### ✨ 반환 값의 포맷을 문서화하는 경우

#### 💡 장점

표준적이고, 명확하고, 사람이 읽을 수 있는 데이터 객체로 저장이 가능하다.

#### 💡 단점
- 그 포맷에 얽매이게 된다.
<br/> -> 이를 사용하는 프로그래머들이 그 포맷에 맞춰 파싱하고, 새로운 객체를 만들고, 영속 데이터로 저장하는 코드를 작성한다. 따라, 다수의 사용자들로부터 포맷이 얽매여 있기 때문에 변경 시 고충을 겪을 것이다.

#### 💡 코드 예시


```java
/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * ...
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면, 앞에서부터 0으로 채워나간다.
 */

@Override public String toString() {
  return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

### ✨ 반환 값의 포맷을 문서화하지 않는 경우

- 유연성
<br/> ->향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있다.

#### 💡 코드 예시

```java
/**
 * 이 약물에 관한 대략적인 설명을 반환한다.
 * 다음은 이 설명의 일반적인 형태이나, 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
 * 
 * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
 */
@Override public String toString() {
  ...
}
```

#### 💡 포맷 명시 여부와 관계 없이, 의도는 명학하게 밝혀야 한다.

또한, toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다. 그렇지 않으면 toString의 반환값을 파싱 해야 하고, 이는 성능 저하의 문제로 이어질 수 있다.