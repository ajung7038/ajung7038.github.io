---
title: "[Java] Item 60 : 정확한 답이 필요하다면 float와 double은 피하라"
categories:
  - Effective Java
tags:
toc: true
toc_sticky: true
date: 2025-05-16 13:57:00 +0900
---

<strong>Effective Java를 읽고 정리한 정리본입니다.</strong>
{: .notice}

# 📌 Item 60 : 정확한 답이 필요하다면 float와 double은 피하라

## 🫧 금융 관련 계산 시 BigDecimal, int, long을 사용하자

float와 double 타입은 특히 금융 관련 계산과는 맞지 않는다. 0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없기 때문이다. (언더플로우)

따라서 BigDecimal, int 혹은 long을 사용하는 게 좋다.

다만, BigDecimal은 단점이 두 가지가 존재한다.

1. 기본 타입보다 쓰기가 훨씬 불편하다.
2. 느리다

그 대안으로 int 혹은 long 타입이 있다.

그럴 경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.