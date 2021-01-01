---
layout: posts
title: '백준 5525번: IOIOI'
date: 2020-12-02 18:02:25
tags:
  - 알고리즘
  - KMP
  - 문자열
  - 백준
---

## 문제

[https://www.acmicpc.net/problem/5525](https://www.acmicpc.net/problem/5525)
* * *

## 코드

```java
public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    int n = Integer.parseInt(br.readLine());
    br.readLine();

    int answer = solution(n, br.readLine());

    System.out.println(answer);
}

public static int solution(int n, String target) {
    int answer = 0;

    char[] s = target.toCharArray();
    char[] targetString = getTargetString(n).toCharArray();
    int[] fail = failFunction(targetString);

    int start = 0;
    int m = 0;

    while (start <= s.length - targetString.length) {
        if (m < targetString.length && s[start + m] == targetString[m]) {
            ++m;
            if (m == targetString.length) {
                answer++;
            }
        } else {
            if  (m == 0) {
                start++;
            } else {
                start += (m - fail[m - 1]);
                m = fail[m - 1];
            }
        }
    }

    return answer;
}

private static String getTargetString(int n) {
    StringBuilder sb = new StringBuilder();
    sb.append("I");

    while (n-- > 0) {
        sb.append("OI");
    }
    return sb.toString();
}

private static int[] failFunction(char[] target) {
    int n = target.length;
    int[] fail = new int[n];

    int start = 1;
    int m = 0;

    while (start + m < n) {
        if (target[start + m] == target[m]) {
            m++;
            fail[start + m - 1] = m;
        } else {
            if (m == 0) {
                start++;
            } else {
                start += (m - fail[m - 1]);
                m = fail[m - 1];
            }
        }
    }

    return fail;
}
```

* * *

## 흐름

* __[KMP 알고리즘](https://ko.wikipedia.org/wiki/%EC%BB%A4%EB%88%84%EC%8A%A4-%EB%AA%A8%EB%A6%AC%EC%8A%A4-%ED%94%84%EB%9E%AB_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)__ 을 이용해 해결

1. S에서 찾을 문자열(targetString)을 만든다.
2. 찾을 문자열에서 실패함수 값을 구한다.
3. 배열의 범위를 벗어나기 전까지 반복한다.
4. 검색 할 문자열 s에서 찾을 단어을 한 문자씩 비교한다.
5. 한 문자가 같으면, 그 다음 문자를 비교 할 수 있게 m을 1씩 더해서 최종적으로 문자열에 단어가 포함되어 있으면 answer를 증가시킨다.
6. 문자가 일치하지 않으면
   1. m이 0이면 시작도 못해본거니깐 검색 할 문자열에서 한 문자 뒤로 간다.
   ex) S = ABCDEF 일 경우, B부터 검색하도록 start를 증가
   2. 0이 아니면 단어에서 어느정도 일치한 문자열이 존재한 것이므로 비교 시작 할 문자의 index를 m에서 실패함수 값을 빼서 구한다.
   3. 일치한 문자열 뒤 부터 검색하면 되므로 m도 실패함수 값에서 구한다.
7. 반복이 끝나면 answer 값을 return 한다.
8. 끝.
* * *

## KMP 알고리즘 해설

{% blockquote 위키백과 %}
문자열에서 특정 패턴을 찾아내는 문자열 검색 알고리즘
{% endblockquote %}

* 비교한 정보를 최대한 활용
* 문자열 S = OOIOIIOIOIOIOIOIOIOIOOIOI 에서 패턴 M = IOIOI 를 찾는다고 가정
* 우선 패턴 M의 실패함수를 구하기 위해 IOIOI에서 접두사와 접미사의 길이가 가장 긴 부분을 구한다.

### 실패함수

```java
private static int[] failFunction(char[] target) {
    int n = target.length;
    int[] fail = new int[n];

    int start = 1;
    int m = 0;

    while (start + m < n) {
        if (target[start + m] == target[m]) {
            m++;
            fail[start + m - 1] = m;
        } else {
            if (m == 0) {
                start++;
            } else {
                start += (m - fail[m - 1]);
                m = fail[m - 1];
            }
        }
    }

    return fail;
}
```

1. IOIOI 패턴에서 index 1에 저장된 'O'와 0에 저장된 'I'는 같지 않으므로 비교가 시작되는 위치를 1 증가 시킨다. (현재 start = 2, m = 0)
2. 루프에 의해 다시 비교하면 2에 저장된 'I'와 0에 저장된 'O'는 같으니 m을 증가시키고 접두사와 접미사가 일치한 값인 m를 저장한다. (현재 start = 2, m = 1)
   1. IOIOI에서 IOI까지 진행한 상태에서 I와 I가 일치 했으므로 길이 1이 일치한 것
3. 3에 저장된 'O'과 1에 저장된 'O'가 같으므로, 다시 m을 1 증가 시키고 접두사와 접미사가 일치한 값인 m을 저장한다. (현재 start = 2, m = 2)
   1. IOIOI에서 IOIO까지 진행한 상태에서 'IO'와 'IO'가 같으므로 길이 2가 일치한 것
4. 4에 저장된 'I'과 2에 저장된 'I'가 같으므로, 다시 m을 1 증가 시키고 접두사와 접미사가 일치한 값인 m을 저장한다. (현재 start = 2, m = 3)
   1. IOIOI는 접두사 'IOI'와 'IOI'의 길이가 3이므로 3
5. array return
6. 끝

### KMP 알고리즘

* 실패함수 값 구하는 것과 동일하다.

```java
public static int solution(int n, String target) {
    int answer = 0;

    char[] s = target.toCharArray();
    char[] targetString = getTargetString(n).toCharArray();
    int[] fail = failFunction(targetString);

    int start = 0;
    int m = 0;

    while (start <= s.length - targetString.length) {
        if (m < targetString.length && s[start + m] == targetString[m]) {
            ++m;
            if (m == targetString.length) {
                answer++;
            }
        } else {
            if  (m == 0) {
                start++;
            } else {
                start += (m - fail[m - 1]);
                m = fail[m - 1];
            }
        }
    }

    return answer;
}
```

* 실패함수 array [0, 0, 1, 2, 3]

1. m이 패턴의 길이보다 작고, 문자열 s(OOIOIIOIOIOIOIOIOIOIOOIOI) 에서 start+m 한 index의 문자와 비교할 패턴(IOIOI)의 m 번째 문자가 같지 않으므로 s에서 비교가 시작되는 위치를 1 증가 시킨다. (현재 start = 1, m = 0)
2. S의 1와 패턴의 0을 비교해도 여전히 같지 않으므로 시작 위치를 또 증가 시킨다. (start = 2, m = 0)
3. S의 2와 패턴의 0을 비교하면 둘다 'I'로 같으므로 m을 증가 시키고 패턴의 길이와 같을 때 까지 반복한다. (start = 2, m = 1)
4. S의 3와 패턴의 1을 비교하면 둘다 'O'로 같으므로 m을 증가 시키고 패턴의 길이와 같을 때 까지 반복한다. (start = 2, m = 2)
5. S의 4와 패턴의 2을 비교하면 둘다 'I'로 같으므로 m을 증가 시키고 패턴의 길이와 같을 때 까지 반복한다. (start = 2, m = 3)
6. S의 5와 패턴의 3을 비교하면 같지 않고 이제 m이 0이 아니므로,
7. s의 시작 위치를 증가 시켜야 하는데 1을 증가 시키는게 아니라 이전에 구한 실패 함수 값에서 찾아온다.
   1. (m(3) - 실패함수 값 array[m(3) - 1]) = 2
   2. 즉 현재 시작 위치에서 1을 더한 값이 아닌 2를 더한다.
   3. 이미 index 4까지는 접두사 접미사가 2 자리까지 같기 때문에
8. m 역시 실패함수 값을 가져온다.
   1. IOIOI에서 IOIO까지 비교한 값에서 실패했기 떄문에 
   2. 그 이전 접미사 접두사 길이만큼으로 비교한다.
9. 이 시점에서 start = 4, m = 1이 되고 루프를 반복한다.
10. 4와 1은 같지 않으므로 다시 7번부터 8번까지 반복해서 start와 m을 조정한다.
    1. 이 예제로 하면 처음부터 같지 않았으므로 start는 한칸만 뒤로 가고
    2. m 역시 접두사 접미사 1 짜리도 실패했으므로 처음부터 비교한다.
    3. (start = 5, m = 0)
11. (5, 0), (6, 1), (7, 2), (8, 3), (9, 4) 이 쭉~ 같으므로 드디어 문자열 s에서 패턴을 찾은 것이므로 answer을 증가 시킨다.
12. 그 이후로 다시 비교하면 m의 길이가 패턴의 길이를 넘어 갔으므로,
13. start와 m의 위치를 다시 조정한다.
14. 반복
15. 끝.

* 결국 정리하면, 처음 말한 것 처럼 이미 비교한 값은 다시 비교하지 않고 그 다음 부터 비교하는 방식으로 시간을 줄인다.
* 더 자세한 설명들은 글 마지막 참고 사이트를 찾아가면 다른 분들이 매우 자세하게 설명해주시고 계신다.

* * *

## 결과

{% asset_img 캡처.PNG 백준 5525번: IOIOI %}
* * *

## 테스트 케이스

```java
assertEquals(4, test.solution(1, "OOIOIOIOIIOII"));
assertEquals(6, test.solution(2, "OOIOIIOIOIOIOIOIOIOIOOIOI"));
assertEquals(7, test.solution(1, "IOIOIOIOIOIOIOI"));
```

## 참고 사이트

* [https://m.blog.naver.com/tpinlab/10119424299](https://m.blog.naver.com/tpinlab/10119424299)
* [https://blog.naver.com/PostView.nhn?blogId=kks227&logNo=220917078260&categoryNo=299&parentCategoryNo=0&viewDate=&currentPage=4&postListTopCurrentPage=&from=menu&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=4](https://blog.naver.com/PostView.nhn?blogId=kks227&logNo=220917078260&categoryNo=299&parentCategoryNo=0&viewDate=&currentPage=4&postListTopCurrentPage=&from=menu&userTopListOpen=true&userTopListCount=5&userTopListManageOpen=false&userTopListCurrentPage=4)
* [https://baeharam.github.io/posts/algorithm/kmp/](https://baeharam.github.io/posts/algorithm/kmp/)
* [https://vvshinevv.tistory.com/2](https://vvshinevv.tistory.com/2)
