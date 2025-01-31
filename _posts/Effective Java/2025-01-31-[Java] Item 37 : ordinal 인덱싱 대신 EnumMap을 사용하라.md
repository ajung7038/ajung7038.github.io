---
title: "[Java] Item 37 : ordinal 인덱싱 대신 EnumMap을 사용하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-01-31 11:56:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 37 : ordinal 인덱싱 대신 EnumMap을 사용하라

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다.

여기서 ordinal 메서드란 모든 열거 타입에 존재하는 메서드로, 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환한다.

자세한 내용은 [item 35](https://ajung7038.github.io/effective%20java/Java-Item-35-ordinal-%EB%A9%94%EC%84%9C%EB%93%9C-%EB%8C%80%EC%8B%A0-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%ED%95%84%EB%93%9C%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/)를 참조하라.

식물을 간단히 나타낸 다음 클래스를 예로 살펴보자.

```java
class Plant {
  enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

  final String name;
  final LifeCycle lifeCycle;

  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }

  @Override
  public String toString() {
    return name;
  }
}
```

이 코드의 목적은 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기 (한해살이, 여러해살이, 두해살이) 별로 묶는 것이다.

목적을 달성하기 위해서는 아래와 같은 순서로 진행이 필요하다.

1. 생애주기별 총 3개의 집합을 만든다.
2. 정원 (Plant가 포함된 리스트 형태, 코드에서는 garden으로 표현) 을 순회한다.
3. 각 식물의 생애주기를 확인한다.
4. 해당 식물의 생애주기와 일치하는 생애주기별 집합 중 하나 (한해살이, 여러해살이, 두해살이 중 하나)를 찾아 넣는다.

이러한 내용은 다음과 같은 두 가지 방식의 코드로 구현이 가능하다.

1. ordinal()을 배열 인덱스로 사용
2. EnumMap을 사용 (데이터 - 열거 타입 매핑)

### ✨ ordinal을 사용한 예시 코드

```java

// 1. 생애주기별 총 3개의 집합을 만든다.
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i=0; i<plantsByLifeCycle.length; i++>) {
  plantsByLifeCycle[i] = new HashSet<>();
}

// 2. 정원 (Plant가 포함된 리스트 형태, 코드에서는 garden으로 표현) 을 순회한다.

// 3. 각 식물의 생애주기를 확인한다. -> p.lifeCycle.ordinal()

// 4. 해당 식물의 생애주기와 일치하는 생애주기별 집합 중 하나 (한해살이, 여러해살이, 두해살이 중 하나)를 찾아 넣는다. -> .add(p);
for (Plant p : garden) {
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

// 결과 출력
for (int i=0; i<plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

그러나 코드에서 볼 수 있듯, 문제는 한가득이다.

1. 배열은 제네릭과 호환되지 않는다. ([item 28](https://ajung7038.github.io/effective%20java/Java-Item-28-%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC/))
2. 배열과 인덱스 사이의 연관성이 없어 출력 결과에 레이블을 달지 않으면 어떤 뜻인지 파악이 어렵다.
3. 타입 안전하지 않다.
<br /> -> 정확한 정숫값을 사용하지 않으면 잘못된 동작을 수행하거나 예외가 터진다.

### ✨ EnumMap을 사용 (데이터 - 열거 타입 매핑)

이러한 문제를 해결하기 위해 EnumMap이 등장했다.

배열은 실질적으로 열거 타입 상수를 값으로 매핑시키는 역할을 하므로, 배열 대신 Map을 사용해도 충분히 같은 효과를 낼 수 있다는 점을 활용하면 EnumMap으로 해당 코드를 구현할 수 있다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

// 1. 생애주기별 총 3개의 집합을 만든다.
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());

// 2. 정원 (Plant가 포함된 리스트 형태, 코드에서는 garden으로 표현) 을 순회한다.

// 3. 각 식물의 생애주기를 확인한다. -> p.lifeCycle

// 4. 해당 식물의 생애주기와 일치하는 생애주기별 집합 중 하나 (한해살이, 여러해살이, 두해살이 중 하나)를 찾아 넣는다. -> plantsByLifeCycle.get(p.lifeCycle).add(p);

for (Plant p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다.

특히, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 필요도 없다.

성능이 떨어지지 않는 이유는 그 내부에서 배열 (ordinal) 을 사용하기 때문이다.

ordinal을 사용하는 것과의 차이점은 내부 구현 방식을 그대로 드러냈냐 숨겼냐의 차이에 있다.

EnumMap에서 내부 구현 방식을 안으로 숨김으로써 타입 안전성과 배열의 성능을 모두 얻어냈다.

또한, 코드 중 유심히 보아야 할 부분이 한 곳 더 있다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
```

여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 <strong>한정적 타입 토큰</strong>으로, 런타임 제네릭 타입 정보를 제공하는 역할을 한다.

자세한 내용은 item 33을 참조하기 바란다.


## 🫧 스트림 사용 코드

이 뿐만 아니라 EnumMap을 사용하지 않고 똑같은 효과를 낼 수 있는 한 가지의 방법이 더 존재한다.

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

EnumMap은 언제나 식물의 생애주기당 하나씩 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다는 차이가 있다.

따라, 배열의 인덱스를 얻기 위해 ordinal을 쓰는 대신, EnumMap을 사용해라. 스트림 사용 또한 괜찮다.