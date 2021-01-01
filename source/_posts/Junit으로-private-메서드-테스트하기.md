---
layout: posts
title: Junit으로 private 메서드 테스트하기
date: 2020-06-23 18:45:14
tags:
    - Junit
    - Test
---

## 서론

[private method도 테스트 해야되는지](https://blog.shiren.dev/2020-06-15-%EC%9C%A0%EC%9A%A9%ED%95%9C%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%BC%80%EC%9D%B4%EC%8A%A4%EB%A5%BC%EC%9C%84%ED%95%9C%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%9D%98%EC%9E%90%EC%84%B8/)에 대해선 의견이 많은 것으로 알고 있지만 현재 프로젝트에서 service에서 있는 메서드 중 주요 로직은 private method에 존재하고 public method에선 호출해서 return만 해주는 method가 존재했는데, 이런 경우에 private 메서드를 테스트하기 위한 방법을 정리해둔다.

* * *

## 본론

```java
@Test
void getPassAndFailWordList_성공()  throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    //given
    String[] answerIds = new String[] {"6_1", "4_1", "5_1", "9_3"};
    WordService w = new WordServiceImpl(repository, userService);

    Method method = w.getClass().getDeclaredMethod("getPassAndFailWordList", String[].class);
    method.setAccessible(true);

    List<Integer> passWordList = Arrays.asList(6,4,5);
    List<Integer> failWordList = Arrays.asList(9);

    Object[] obj = new Object[] {answerIds};

    //when
    Map<String, List<Integer>> map = (Map<String, List<Integer>>) method.invoke(w, obj);

    //than
    assertThat(map).extracting("pass", String.class)
            .contains(passWordList);
    assertThat(map).extracting("fail", String.class)
            .contains(failWordList);
}
```

1. test할 private method가 존재하는 class를 직접 생성
2. [getDeclaredMethod()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-)를 이용해서 해당 클래스에 존재하는 private method를 가져오고
3. [setAccessible()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AccessibleObject.html#setAccessible-java.lang.reflect.AccessibleObject:A-boolean-)로 private method에 접근을 허용
4. [invoke()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#invoke-java.lang.Object-java.lang.Object...-)로 호출하는데 이 때 invoke()의 매개변수가 Object[] 이므로 원래 호출하려던 private method의 매개변수를 Object[]에 담은 후 Object[]을 매개변수로 넘겨줘야함

* * *

## 참고 사이트

- [https://stackoverflow.com/questions/34571/how-do-i-test-a-private-function-or-a-class-that-has-private-methods-fields-or](https://stackoverflow.com/questions/34571/how-do-i-test-a-private-function-or-a-class-that-has-private-methods-fields-or)
- [https://www.crocus.co.kr/1665](https://www.crocus.co.kr/1665)
- [https://stackoverflow.com/questions/8189782/wrong-number-of-arguments-error-when-invoking-a-method](https://stackoverflow.com/questions/8189782/wrong-number-of-arguments-error-when-invoking-a-method)