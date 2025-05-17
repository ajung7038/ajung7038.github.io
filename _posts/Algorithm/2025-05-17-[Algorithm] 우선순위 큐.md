---
title: "[Algorithm] 우선순위 큐"
categories:
  - Algorithm
tags:
toc: true
toc_sticky: true
date: 2025-05-17 16:59:00 +0900
---

## PriorityQueue

PriorityQueue란 들어오는 순서에 상관 없이 우선순위에 따라 자동 정렬되는 큐 구조로, 정렬하여 문제를 푸는 경우 많이 사용된다.

- 시간복잡도 O(NlogN) 소요

## PriorityQueue 선언

### 1. 기본 (오름차순/내림차순)

```java
import java.util.PriorityQueue

// 오름차순 정렬
PriorityQueue<자료형> pq = new PriorityQueue<>();

// 내림차순 정렬
PriorityQueue<자료형> pq = new PriorityQueue<>(Collections.reverseOrder());
```

### 2. 기본 메서드

- 원소 추가
    - add() : 실패 시 에러
    - offer() : 실패 시 false
- 원소 삭제
    - poll() : 실패 시 null
    - remove() : 실패 시 에러
- 원소 확인
    - peek() : 첫 번째 값 반환, 제거 X, 비어있다면 null
    - element() : 첫 번째 값 반환, 제거 X, 비어있다면 예외
- 기타
    - isEmpty() : 실패 시 에러
    - clear() : 초기화
    - size() : 원소 수 반환

### 3. 객체 우선순위 정하기
기본 타입 형에서는 위와 같이 오름차순, 내림차순으로 구현이 가능하다.

그런데 객체의 경우 크고 작음을 어떻게 확인하는 것일까?

```java
import java.util.PriorityQueue;

public class Example {
    private class Student {
        int mathScore; // 수학점수
        int engScore;  // 영어점수
        Student(int mathScore, int engScore){
            this.mathScore = mathScore;
            this.engScore = engScore;
        }
    }
    public static void main(String[] args) {
        PriorityQueue<Student> pQ = new PriorityQueue<>();
    }
}
```

다음과 같이 Student를 우선순위 큐에 삽입하려고 한다.

만일 수학점수가 높은 학생이 우선순위가 높다고 판단하고, 수학 점수가 같은 경우 영어 점수가 높은 학생이 우선순위가 높다고 판단한다면 다음과 같이 코드를 작성할 수 있을 것이다.

```java
import java.util.Comparator;
import java.util.PrirityQueue;

public class Example {
    // inner class
    private class Student {
        int mathScore; // 수학점수
        int engScore;  // 영어점수
        Student(int mathScore, int engScore){
            this.mathScore = mathScore;
            this.engScore = engScore;
        }
    }

    PriorityQueue<Student> pQ = new PriorityQueue<>(new Comparator<Student>() {
        @Override
        public int compare(Student o1, Student o2) {
            if (o1.mathScore == o2.mathScore)
                return o2.engScore - o1.engScore;
            return o2.mathScore - o1.mathScore;
        }
    });
}
```

이때 람다식으로 더 간단하게 작성이 가능하다.

```java
PriorityQueue<Student> pQ = new PriorityQueue<>((o1, o2) -> {
    if (o1.mathScore == o2.mathScore)
        return o2.engScore - o1.engScore;
    return o2.mathScore - o1.mathScore;
});
```

또는 더 짧게 줄여,

```java
PriorityQueue<Student> pQ = new PriorityQueue<>((o1, o2) -> o1.mathScore == o2.mathScore ? o2.engScore - o2.engScore : o2.mathScore - o1.mathScore);
```

성능 차이는 거의 없으며, 오히려 람다식이 더 가독성이 좋아 많이 사용하는 추세이다.

## 참고 자료
- https://kbj96.tistory.com/49