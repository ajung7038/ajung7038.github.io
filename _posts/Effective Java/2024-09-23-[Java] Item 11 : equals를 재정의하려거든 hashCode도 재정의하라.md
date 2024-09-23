---
title: "[Java] Item 11 : equals를 재정의하려거든 hashCode도 재정의하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2024-09-23 12:47:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 11 : equals를 재정의하려거든 hashCode도 재정의하라

## 🫧 equals와 hashCode

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

### ✨ HashCode 관련 Object 명세
1) equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2) equals(Object) 가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3) equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.


### ✨ hashCode 재정의를 잘못 했을 때의 코드 예시

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```

m.get(new PhoneNumber(707, 867, 5309))를 반환하면 어떤 값이 나올까?

"제니"를 기대했지만, 실제로는 null이 반환되게 된다.

이러한 문제가 생기는 이유는 논리적 동치인 두 객체를 다르다고 판단했기 때문이다. 따라, hashCode를 올바르게 재정의 해야 이러한 문제가 생기지 않는 것이다.

### ✨ 좋은 hashCode 작성하기

1) int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다
<br/> -> equals 비교에 사용되는 필드를 핵심 필드로 지칭한다.

2) 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    
    a. 해당 필드의 해시코드 c를 계산한다.

        i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
        ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
        iii. 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다.

        -> 만약 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

    b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다.
    <br /> -> result = 31 * result + c;

3) result를 반환한다.

💡 파생 필드는 해시코드 계산에서 제외해도 되며, equals 비교에 사용되지 않은 필드는 '반드시' 제외해야 hashCode 규약 두 번째를 어기지 않을 수 있다.


### ✨ 코드 예시

```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    retsult = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

같은 기능을 하는 코드이지만, 두 번째 코드는 첫 번째 코드에 비해 성능이 그렇게 좋지 않다.

또한, 해시코드를 지연 초기화하는 hashCode 메서드 또한 구현할 수 있다.

이 방식은 "캐싱"을 사용하는 방식으로, 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 도입할 만하다.

```java
@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        retsult = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

또한, hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말아야지 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.