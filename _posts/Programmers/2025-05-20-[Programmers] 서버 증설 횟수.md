---
title: "[Programmers] 서버 증설 횟수"
categories:
  - Programmers
tags:
toc: true
toc_sticky: true
date: 2025-05-20 14:37:00 +0900
---

## 문제
[프로그래머스-서버 증설 횟수](https://school.programmers.co.kr/learn/courses/30/lessons/389479)

- 하루 24시간 동안 매 시간마다 요청 수(players[i])가 주어진다.
- 서버 한 대는 m명의 사용자 요청 처리 가능
- 서버는 증설 후 k시간 동안 유지되고 이후 자동으로 사라짐.

## 풀이 1 - 배열
```java
class Solution {
    public int solution(int[] players, int m, int k) {
        int count = 0; // 증설 횟수
        int[] server = new int[24]; // 증설된 서버 수 저장 배열
        
        for (int i=0; i<players.length; i++) {
            int n=server[i];  // 현재 서버 수
            if (players[i] >= (n+1) * m) {
                // 증설
                int add = (players[i]-(m*n)) / m;
                n += add;
                
                // 시간이 지나면 증설한 서버가 사라짐
                for (int j=i; j<i+k && j<24; j++) {
                    server[j] += add;
                }
                count += add;
            }
        }
        
        return count;
    }
}
```

- 증설된 서버 수를 배열에 저장해 구현

그러나 다른 사람들의 풀이를 찾아보니, 배열보다는 우선순위 큐가 문제를 풀기에 더 적합하다고 느꼈다.

따라서 다른 분이 올려주신 우선순위 큐 코드를 참조해 문제를 다시 확인하고자 한다.

## 풀이 2 - 우선순위 큐
```java
import java.util.*;
class Solution {
    public int solution(int[] players, int m, int k) {

        // 만료 시간이 빠른대로 정렬
        // pq 안 int[] 는 {만료 시각, 서버 수} 형태를 띔
        PriorityQueue<int[]> pq = new PriorityQueue<>((o1,o2) -> o1[0] - o2[0]);
        int size = 0;  // 현재 서버의 개수 
        int count = 0; // 증설된 서버 횟수  
        for(int i = 0; i < 24; i++){
            // 만료된 서버 내리기
            // pq.peek()[0] == i이면 i시각에 만료되는 서버가 존재한다는 의미
            while(!pq.isEmpty() && pq.peek()[0] == i){
                size -= pq.poll()[1];
            }
            int need = players[i] / m;  // 현재 필요한 서버의 개수 
            int more = size - need;     // - 서버 증설 개수  
            if(more < 0){
                more = -more; // more가 음수면 부족한 서버 수이므로 -more로 변환
                size  += more;
                count += more;
                pq.add(new int []{i + k, more});
            }
        }
        return count;
    }
}
```

## 참고 자료
- https://20240228.tistory.com/436