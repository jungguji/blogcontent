---
layout: posts
title: 'LeetCode 1829. Maximum XOR for Each Query'
date: 2023-01-04 22:07:37
tags:
    - 알고리즘
    - LeetCode
    - Prefix sum
---
## 문제

[https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/](https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/)

 정렬된 n개의 음의 아닌 정수 배열 nums와 정수 maximumBit가 주어지고, 아래 쿼리를 n번 수행한다.
 
 > nums [0] XOR nums [1] XOR ... XOR nums [nums.length-1] XOR k

<br />

 를 최대화하는 k <2maximumBit의 음이 아닌 정수를 찾는데 이 때,  k는 각 쿼리의 정답이다.
  쿼리를 반복할 때 마다 배열 nums의 마지막 element를 제거하고, answer 배열을 반환하는데, 여기서 answer[i]는 i번째 쿼리의 정답 즉, k 이다.

* * *

## 코드

```java
public int[] getMaximumXor(int[] nums, int maximumBit) {
    int[] pxor = getPrefixXOR(nums);

    int[] answer = new int[nums.length];

    int answerIndex = 0;
    int maxValue = (1 << maximumBit) - 1;

    for (int i = pxor.length-1; i > 0; --i) {

        answer[answerIndex++] = pxor[i] ^ maxValue;
    }

    return answer;
}

private int[] getPrefixXOR(int[] nums) {
    int[] psum = new int[nums.length + 1];

    for (int i = 1; i <= nums.length; ++i) {
        psum[i] = psum[i-1] ^ nums[i-1];
    }
    return psum;
}
```

* * *

## 흐름

1. 누적합(prefix sum) 알고리즘을 통해 누적 XOR 배열을 구한다.
2. 이 때, 2의 maximumBit승보다 작은 수 중에 가장 큰 수는 당연히 (2^maximumBit)-1 한 수 이므로, (2^maximumBit)-1 값을 변수에 할당해놓고,
3. nums.length의 길이인 N 번 만큼 쿼리를 반복하면서 마지막 원소를 제거하면서 k 값을 구해 answer[i]에 저장한다.
4. 끝.

## pxor[i] ^ maxValue가 K인 이유

```java
void xorTEST() {
    int x = 93;
    int y = 12;
    int z = x ^ y;

    System.out.println("----- 초기값 -----");
    System.out.println("x = " + x);
    System.out.println("y = " + y);
    System.out.println("z = " + z);

    System.out.println("----- XOR -----");
    System.out.printf("x ^ y = %d(z)\n", x^y);
    System.out.printf("x ^ z = %d(y)\n", x^z);
    System.out.printf("y ^ z = %d(x)\n", y^z);
}
```

```text
----- 초기값 -----
x = 93
y = 12
z = 81
----- XOR -----
x ^ y = 81(z)
x ^ z = 12(y)
y ^ z = 93(x)
```

 우리가 구해야 하는 값은 쿼리의 값을 가장 크게 할 값인 K이고, 우리는 이미 가장 큰 값을 (2^maximumBit)-1 로 구해놓았으므로,
누적합 알고리즘으로 구한 XOR값에 가장 큰 값을 XOR하면 k가 나오게 된다.


* * *

## 결과

{% asset_img 캡처.PNG %}
* * *