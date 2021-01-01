---
layout: posts
title: BDDMockito given 사용시 주의
date: 2020-06-09 13:04:42
tags:
    - Spring-Boot
    - Junit
    - Test
---

## 문제

### Test Code

```java
@Test
void testCreateWordFail() throws Exception {
    // gvien
    BindingResult bindingResult = new MapBindingResult(new HashMap(), "");
    bindingResult.rejectValue("test", "test", "haha");

    WordDTO.AddWord addWord = new WordDTO.AddWord();
    addWord.setWord("test");
    addWord.setMeaning("테스트");

    given(this.service.getCreateWordBindingResult(addWord, bindingResult)).willReturn(bindingResult);

    String addWordToJson = getAddWordToJson();

    //when
    ResultActions action = mockMvc.perform(post("/word/add")
            .param("save", "")
            .content(addWordToJson)
            .contentType(MediaType.APPLICATION_JSON)
            .with(csrf()))
            .andDo(print());

    //then
    action.andExpect(status().isOk())
            .andExpect(view().name("thymeleaf/word/createWordForm"));
}
```

위 코드처럼 this.service.getCreateWordBindingResult(addWord, bindingResult) method가 호출될 때 willdReturn으로 지정한 bindingResult가 return되게 하고 하려했으나 given이 제대로 동작하지 않아서 인지 getCreateWordBindingResult method 안에서 result가 존재하지 않아 NullPointerException이 발생하여 테스트가 실패했다.

### Test Target

```java
@PostMapping(value="/word/add", params= {"save"})
public String createWord(@Valid AddWord word, BindingResult bindingResult) {
    BindingResult result = service.getCreateWordBindingResult(word, bindingResult);

    if (result.hasErrors()) { // NullPointerException 발생
        return "thymeleaf/word/createWordForm";
    }

    service.insertWord(word);
    return "thymeleaf/index";
}
```

* * *

## 해결

ArgumentMatchers class에 [any()](https://javadoc.io/doc/org.mockito/mockito-core/2.2.7/org/mockito/ArgumentMatchers.html#any(java.lang.Class))를 이용해서 파라미터를 넘겼더니 정상적으로 동작했다.

```java
given(this.service.getCreateWordBindingResult(any(WordDTO.AddWord.class), any(BindingResult.class))).willReturn(bindingResult);
```

- p.s : 기본 자료형 (int, char)이나 String 등은 any()를 사용하지 않고 그대로 파라미터로 받아도 정상적으로 동작하는 것 같은데 어째서 따로 정의한 클래스만 인식(?) 하지 못해서 willReturn이 먹히지 않고 null이 return 되는지 알 수가 없었다.

* * *

## 전체 코드

- [https://github.com/jungguji/wordbook](https://github.com/jungguji/wordbook)