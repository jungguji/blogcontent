---
layout: posts
title: '프로그래머스: H-Index'
date: 2020-05-08 17:42:01
tags:
    - 프로그래머스
    - 알고리즘
    - 정렬
---

## 문제

[https://programmers.co.kr/learn/courses/30/lessons/42747?language=java](https://programmers.co.kr/learn/courses/30/lessons/42747?language=java)

* * *

## 코드

```java
public int solution(int[] citations) {
    int answer = 0;
    int n = citations.length;

    Arrays.sort(citations);

    for (int i = 0; i <= citations[n - 1]; i++) {
        int high = 0;

        for (int j = 0; j < n; j++) {
            if (i <= citations[j]) {
                high++;
            }

            if (i <= high) {
                answer = i;
            }
        }
    }

    return answer;
}
```

* * *

## 흐름

1. 논문을 인용한 횟수가 담긴 배열 citations를 정렬

2. 가장 많이 인용한 횟수 만큼 반복

3. 0부터 배열 길이만큼 반복하면서 i 값이 인용 횟수보다 작은 경우 i 보다 인용된 횟수가 많은 것이므로 high 변수 증가

4. h번 이상 인용된 논문이 h편 이상이여야 하므로 i가 high 보다 작을 경우에만 answer에 저장

5. i가 high보다 커질 경우 answer에 저장되지 않으므로 answer에 마지막으로 저장된 값이 h

6. 끝

* * *

## 다른 해결방법

### 코드

```java
public int solution(int[] citations) {
    Arrays.sort(citations);

    int max = 0;
    for(int i = citations.length-1; i > -1; i--){
        int min = (int)Math.min(citations[i], citations.length - i);
        if(max < min) max = min;
    }

    return max;
}
```

### 위 코드의 로직

1. 배열을 정렬 후 배열의 마지막 값과 1 부터 증가 시키면서 citations의 길이 만큼 반복함

2. 예를 들어 {0, 1, 3, 5, 6} 인 경우, citations.length는 5, i = 5 - 1 이므로 4, citations.length - i는 5 - 4가 되므로 1임 그런고로, 1과 citations[4]인 6과 비교해서 작은 놈을 min에 저장

3. 위 과정을 반복하면
min = Min(6,1)
min = Min(5,2)
min = Min(3,3)
min = Min(1,4)
min = Min(0,5)
가 되고 max는 max가 min 보다 작을 때 max에 min 값이 저장되므로 max엔 최종적으로 3이 저장

- 어떻게 유추해서 푼 건지 이해가 안간다 천재인 듯

* * *

## 결과

### 1번

|번호|속도|
|----|----|
|테스트 1 |    통과 (11.43ms, 50.3MB)
|테스트 2 |    통과 (14.71ms, 52.4MB)
|테스트 3 |    통과 (15.13ms, 50.7MB)
|테스트 4 |    통과 (13.74ms, 50.1MB)
|테스트 5 |    통과 (16.49ms, 50.6MB)
|테스트 6 |    통과 (13.86ms, 50MB)
|테스트 7 |    통과 (10.08ms, 52.4MB)
|테스트 8 |    통과 (7.45ms, 52.6MB)
|테스트 9 |    통과 (8.26ms, 52.9MB)
|테스트 10 |    통과 (10.29ms, 50MB)
|테스트 11 |    통과 (17.04ms, 50.9MB)
|테스트 12 |    통과 (8.21ms, 52.8MB)
|테스트 13 |    통과 (15.30ms, 52.5MB)
|테스트 14 |    통과 (13.62ms, 52.7MB)
|테스트 15 |    통과 (15.64ms, 50.5MB)
|테스트 16 |    통과 (0.97ms, 54.7MB)

### 2번

|번호|속도|
|----|----|
|테스트 1 |    통과 (1.10ms, 52.4MB)
|테스트 2 |    통과 (1.36ms, 50.5MB)
|테스트 3 |    통과 (1.36ms, 50.8MB)
|테스트 4 |    통과 (0.85ms, 51.9MB)
|테스트 5 |    통과 (0.95ms, 52.5MB)
|테스트 6 |    통과 (1.35ms, 52.7MB)
|테스트 7 |    통과 (1.00ms, 50.6MB)
|테스트 8 |    통과 (0.87ms, 50.9MB)
|테스트 9 |    통과 (1.06ms, 50.5MB)
|테스트 10 |    통과 (1.11ms, 52.6MB)
|테스트 11 |    통과 (1.39ms, 52.5MB)
|테스트 12 |    통과 (0.91ms, 52.6MB)
|테스트 13 |    통과 (1.22ms, 50.3MB)
|테스트 14 |    통과 (1.50ms, 52.5MB)
|테스트 15 |    통과 (1.36ms, 52.3MB)
|테스트 16 |    통과 (0.94ms, 52.5MB)

- 속도도 압도적...

* * *

## 테스트 케이스

```java
assertEquals(3, test.solution(new int[] {3,0,6,1,5}));
assertEquals(3, test.solution(new int[] {6,5,4,3,1,1,1,1,1}));
assertEquals(3, test.solution(new int[] {0, 1, 1, 1, 1, 3, 3, 4}));
assertEquals(3, test.solution(new int[] {5, 5, 5, 0}));
assertEquals(2, test.solution(new int[] {2,2,2,2,2}));
assertEquals(4, test.solution(new int[] {5, 5, 5, 5}));
assertEquals(5, test.solution(new int[] {5, 5, 5, 5, 5}));
assertEquals(1, test.solution(new int[] {7}));
assertEquals(3, test.solution(new int[] {4, 3, 3, 3, 3}));
assertEquals(0, test.solution(new int[] {0,0,0,0,0,0}));
assertEquals(3, test.solution(new int[] {6, 5, 3, 1, 0}));
```
