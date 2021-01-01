---
layout: posts
title: '프로그래머스: n진수 게임'
date: 2020-08-03 17:03:33
tags:
    - 알고리즘
    - 프로그래머스
---

## 문제

[https://programmers.co.kr/learn/courses/30/lessons/17687](https://programmers.co.kr/learn/courses/30/lessons/17687)
* * *

## 코드

```java
private static final String[] ABCDEF = new String[] {"A","B","C","D","E","F"};
public String solution(int n, int t, int m, int p) {

    int limit = t * m;
    String speakString = make(n, limit);

    char[] toChar = speakString.toCharArray();

    StringBuilder answer = new StringBuilder();

    int index = p - 1;
    for (int i = 0; i < t; i++) {
        answer.append(toChar[index]);
        index += m;
    }

    return answer.toString();
}

public String make(int n, int limit) {
    StringBuilder sb = new StringBuilder().append(0);

    for (int i = 1; i < limit; i++) {
        sb.append(changeDecimal(i, n));
    }

    return sb.toString();
}

public char[] changeDecimal(int currentNumber, int n) {
    StringBuilder sb = new StringBuilder();
    while(currentNumber > 0) {
        int remain = currentNumber % n;
        sb.append(remain >= 10 ? ABCDEF[remain-10] : remain);
        currentNumber /= n;
    }

    return sb.reverse().toString().toCharArray();
}
```

* * *

## 흐름

- 진법 n, 미리 구할 숫자의 갯수 t, 게임에 참가하는 인원 m, 튜브의 순서 p

1. 미리 구할 숫자의 갯수와 참가 인원 수를 곱해서 구해야 할 수의 갯수를 구한다.
2. 1부터 구해야 될 수까지 반복한다.
3. 반복하면서 숫자를 진법으로 나눈 나머지를 구한다.
4. 나머지가 10 이상이면 A, B, C ... 으로 치환한다.
5. 현재 수를 n 진법의 값만큼 나눈다.
6. 작은 수부터 역순으로 저장되어야 하므로 reverse() 메서드로 역순으로 돌린다.
7. 0부터 미리 구할 숫자 갯수만큼 반복하면서
8. n진법으로 치환한 문자열에서 참가인원 숫자만큼 더하면서 숫자를 뽑아 저장한다.
9. 끝

* * *

## 결과

|번호|속도|
|----|----|
|테스트 1 |    통과 (0.87ms, 50.6MB)
|테스트 2 |    통과 (1.15ms, 50.5MB)
|테스트 3 |    통과 (1.02ms, 52MB)
|테스트 4 |    통과 (1.07ms, 52.1MB)
|테스트 5 |    통과 (4.60ms, 52.6MB)
|테스트 6 |    통과 (4.40ms, 52.2MB)
|테스트 7 |    통과 (4.55ms, 53MB)
|테스트 8 |    통과 (2.62ms, 52.3MB)
|테스트 9 |    통과 (2.45ms, 52.6MB)
|테스트 10 |    통과 (2.57ms, 52.3MB)
|테스트 11 |    통과 (2.49ms, 52MB)
|테스트 12 |    통과 (2.45ms, 52.7MB)
|테스트 13 |    통과 (5.32ms, 50.3MB)
|테스트 14 |    통과 (69.61ms, 77.9MB)
|테스트 15 |    통과 (75.19ms, 75.8MB)
|테스트 16 |    통과 (70.55ms, 78.1MB)
|테스트 17 |    통과 (17.09ms, 53.3MB)
|테스트 18 |    통과 (15.42ms, 52.7MB)
|테스트 19 |    통과 (6.46ms, 52.6MB)
|테스트 20 |    통과 (13.17ms, 53MB)
|테스트 21 |    통과 (29.51ms, 58.7MB)
|테스트 22 |    통과 (21.20ms, 56.5MB)
|테스트 23 |    통과 (41.04ms, 64.1MB)
|테스트 24 |    통과 (59.00ms, 77.4MB)
|테스트 25 |    통과 (74.66ms, 80MB)
|테스트 26 |    통과 (24.89ms, 54.3MB)
* * *

## 테스트 케이스

```java
assertEquals("0111", test.solution(2,4,2,1));
assertEquals("02468ACE11111111", test.solution(16,16,2,1));
assertEquals("13579BDF01234567", test.solution(16,16,2,2));
```

## 참고사이트

- [https://tech.kakao.com/2017/11/14/kakao-blind-recruitment-round-3/](https://tech.kakao.com/2017/11/14/kakao-blind-recruitment-round-3/)
