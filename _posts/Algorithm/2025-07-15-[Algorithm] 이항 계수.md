---
title: "[Algorithm] 이항 계수"
categories:
  - Algorithm
tags:
toc: true
toc_sticky: true
date: 2025-07-15 08:38:00 +0900
---

## 이항 계수
- 두 개의 항 (이항) 을 전개하여 계수로 나타낸 것

![nCrImg](../../assets/image/Algorithm/nCr.png)


이는 곱셈 공식으로 계산이 가능하다. => nCr

nCr = n!/r!(n-r)!

## 성질

1. (n+1)C(r+1) = nCr + nC(r+1)
2. nC0 = nCn = 1

이 팩토리얼은 dp를 통해 쉽게 구할 수 있다.

메모이제이션할 배열을 초기에 선언한 후 재귀로 하위 문제를 접근하면 쉽게 중복되는 코드 없이 구할 수 있다.

알고리즘은 다음과 같다.

```java
int[][] dp = new int[n+1][k+1];

int BC(int n, int k) {
    // 이미 구했던 식이라면 (== 메모이제이션 배열에 저장되어 있다면) => 구하지 않고 저장된 값을 활용한다.
    if(dp[n][k] > 0) return dp[n][k];

    // nCn or nC0이라면 배열에 1 저장 후 리턴
    if (n==k || k==0) return dp[n][k]=1;


    return dp[n][k] = BC(n-1)(k-1) + BC(n)(k-1);
}
```



## 참고 자료
[백준 - 11050번 : 이항 계수 1 - JAVA](https://st-lab.tistory.com/159#%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)