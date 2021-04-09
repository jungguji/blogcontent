---
layout: posts
title: '백준 10527번: Judging Troubles'
date: 2021-04-01 22:51:49
tags:
    - 알고리즘
    - 백준
    - 자료구조
---
## 문제

[https://www.acmicpc.net/problem/10527](https://www.acmicpc.net/problem/10527)
* * *

## 코드

```java
public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    int n = Integer.parseInt(br.readLine());

    Map<String, Boolean> isDuplicate = new HashMap<>();
    Map<String, Integer> domJudge = new HashMap<>();

    for (int i = 0; i < n; i++) {
        String input = br.readLine();

        if (!isDuplicate.containsKey(input)) {
            isDuplicate.put(input, false);
        }

        domJudge.put(input, domJudge.getOrDefault(input, 0) + 1);
    }

    Map<String, Integer> kattis = new HashMap<>();

    for (int i = 0; i < n; i++) {
        String input = br.readLine();

        if (isDuplicate.containsKey(input)) {
            isDuplicate.put(input, true);
        }

        kattis.put(input, kattis.getOrDefault(input, 0) + 1);
    }

    List<Map.Entry<String, Integer>> entries1 = new ArrayList<>(domJudge.entrySet());
    List<Map.Entry<String, Integer>> entries2 = new ArrayList<>(kattis.entrySet());

    entries1.addAll(entries2);
    Collections.sort(entries1, (e1, e2) -> {
        if (e1.getKey().compareTo(e2.getKey()) > 0) {
            return 1;
        } else if (e1.getKey().compareTo(e2.getKey()) < 0) {
            return -1;
        } else {
            return Integer.compare(e1.getValue(), e2.getValue());
        }
    });

    int answer = 0;
    Set<String> set = new HashSet<>();
    for (int i = 0; i < entries1.size(); i++) {
        Map.Entry<String, Integer> entry = entries1.get(i);

        if (isDuplicate.getOrDefault(entry.getKey(), false) && set.add(entry.getKey())) {
            answer += entry.getValue();
        }
    }

    System.out.println(answer);
}
```

* * *

## 흐름

- 우선 이 문제는 domJudge와 kattis가 순서대로 n만큼씩 입력 받는 문제다

예제를 보며 얘기하면

```text
5
correct
wronganswer
correct
correct
timelimit
wronganswer
correct
timelimit
correct
timelimit
```

이렇게 되어 있을 때 위에 다섯 줄은 domJudge에서 채점한 결과고, 그 아래 다섯 줄은 kattis 채점한 결과다.

이걸 생각하고 프로그램을 작성하면

1. 우선 양 쪽에서 채점된 결과들 중에 작은 녀석의 값을 골라야 하므로 채점 결과를 Key로 하는 Map을 만들고
2. n 만큼 돌면서 domJudge의 채점 결과를 저장하는데 이 때 처음나온 채점 결과는 Map에 false로 저장하고, 이전에 이미 나온 결과는 +1 해준다.
3. 그 후 다시 n만큼 돌면서 kattis의 채점결과를 저장하는데
4. 이 때는 전에 domJudge에서 나온 채점 결과 인지 확인해야 하므로 중복을 체크하기 위해 만든 Map에 채점 결과가 존재하면 그 key의 값을 true로 변경시켜서 양 쪽에서 모두 나온 결과 임을 저장한다.
5. 그 후 양 쪽에서 모두 나온 채점 결과들 중 작은 값들을 더하기 위해
6. 두 Map을 리스트로 변환 후 합쳐서 key와 value로 sorting 한다.
7. 그 후 합친 리스트를 반복하면서..
8. 채점 결과가 양 쪽 모두에서 나온 녀석이면서 이전에 더한 key가 아니라면 쭉 더하고
9. 출력하고 끝낸다.

* * *

## 결과

{% asset_img 캡처.PNG 백준 10527번: Judging Troubles %}
* * *

## 다른 분의 코드

```java
public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String line = br.readLine().trim();
    int n = Integer.parseInt(line);
    Map<String, Integer> map = new HashMap<>();
    for(int i = 0; i < n; i++){
        String dom = br.readLine();
        map.put(dom, map.getOrDefault(dom, 0) + 1);
    }
    int cnt = 0;
    for(int i = 0; i < n; i++){
        String kattis = br.readLine();
        if(map.containsKey(kattis) && map.get(kattis) > 0){
            cnt++;
            map.put(kattis, map.get(kattis) - 1);
        }
    }
    System.out.println(cnt);
}
```

- 나 처럼 바보 같이 Map을 여러 개 쓰지 않고 하나의 맵에 +1, -1로 해서 겹치는 결과 인지 아닌지 판단했다.
- 나는 왜 이렇게 생각하지 못했을까? 다른 곳에서도 흔히 쓰이는 방식인데
- 오늘도 역시 반성해본다...
