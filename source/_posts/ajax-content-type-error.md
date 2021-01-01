---
layout: posts
title: 'Ajax http status 415 Unsupported Media Type'
date: 2020-05-05 11:31:03
tags:
    - Spring-Boot
    - ajax
---

## 문제

![error]](/images/20200505/image.png)

위 화면 처럼 임시 비밀번호 발급 form을 만들어서 버튼엔 onclick, input box엔 enter key event를 달아서 ajax로 처리하였는데,

​클릭 이벤트로는 정상적으로 동작하였으나 enter key 의 경우 http status 415 Unsupported Media Type 에러가 발생하였다

```java
There was an unexpected error (type=Unsupported Media Type, status=415).
Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported
org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported
 at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodArgumentResolver.readWithMessageConverters(AbstractMessageConverterMethodArgumentResolver.java:225)
 at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.readWithMessageConverters(RequestResponseBodyMethodProcessor.java:158)
 at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.resolveArgument(RequestResponseBodyMethodProcessor.java:131)
 at org.springframework.web.method.support.
    HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:121)
 at org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:167)
...
```

보통 ajax 통신 시 415 에러가 발생 할 경우 ajax에서 설정한 content type과 contoroller에서 설정한 content type이 다를 경우 발생하는 데 이 경우엔 모두 json으로 설정해놓은 상태 하였는데도 불구하고 발생하여 매우 당황스러웠다

### ajax

```javascript

function resetPassword() {
        const userName = document.querySelector("input[name=username]").value.trim();
        const data = { username : userName };

        $.ajax({
            type:"POST"
            , url: "/reset/password"
            , data: JSON.stringify(data)
            , dataType : "json"
            , contentType: "application/json"
            , cache: false
            , success: function(data) {
                let alert = document.querySelector(".mb-0");
                alert.innerText = data;

                document.querySelector(".alert.alert-success").style.display = "block";
            }
        });
    }
```

### Controller

```java
@PostMapping(value="/reset/password", produces = "application/json")
@ResponseBody
public String processResetPasswordForm(@RequestBody User user) throws JsonProcessingException {
    String tempPassword = userService.setTempPassWord(user);
    String msg = "임시 비밀번호 : " + tempPassword;

    System.out.println(msg);
    ObjectMapper objMapper = getObjectMapperConfig();
    return objMapper.writeValueAsString(msg);
}
```

* * *

## 해결

하여 헤매던 도중 enter key event를 지정하지 않아도 이벤트가 발생하여 똑같이 415 에러를 뱉어내는 것으로

확인돼서 적용한 thymeleaf나, bootstrap 등 어디선가 submit 을 보내고 있다고 판단하여 preventDefault() function으로 이벤트를 정지시키고 작성한 ajax를 실행시켰더니 정상적으로 동작하였다

### form의 submit event에 preventDefault()를 추가

```javascript
document.querySelector("#frm").addEventListener('submit', function(e) {
        e.preventDefault();

        resetPassword();
    });
```

### 정상적으로 실행된 모습

![success]]](/images/20200505/image2.png)

* * *

## 전체 코드

```html

<!DOCTYPE html>
<html lang="ko" 
    xmlns="http://www.w3.org/1999/xhtml" 
    xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/config :: configFragment"></head>
<body>
    <div class="container">
        <div class="d-flex justify-content-center h-100">
            <div class="card" style="height:auto;">
                <div class="card-header">
                    <h3>Reset Password</h3>
                </div>
                <div class="card-body">
                    <form id="frm" th:action=@{/reset/password} method="post">
                    <div class="alert alert-success" role="alert" style="display:none;">
                      <h4 class="alert-heading">Well done!</h4>
                      <p>해당 비밀번호는 임시로 생성된 비밀번호 입니다. <br />로그인 후 꼭 비밀번호를 변경하여 주세요.</p>
                      <hr>
                      <p class="mb-0"></p>
                    </div>
                        <div class="alert alert-danger" role="alert" style="float:left; margin-right:1rem; display:none;" ></div>
                        <div class="input-group form-group">
                            <div class="input-group-prepend">
                                <span class="input-group-text"><i class="fas fa-user"></i></span>
                            </div>
                            <input type="text" onkeyup="checkUserName()" class="form-control" name="username" placeholder="username">
                        </div>
                        <div class="form-group">
                            <input type="submit" onclick="resetPassword()" value="확인" class="btn float-right login_btn" disabled>
                        </div>
                    </form>
                </div>
                <div class="card-footer">
                    <div class="d-flex justify-content-center links">
                        Don't have an account?<a href="/login">Sign In</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
    <link th:href="@{/css/login.css}" rel="stylesheet" type="text/css">
    <!--Fontawesome CDN-->
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.3.1/css/all.css" integrity="sha384-mzrmE5qonljUremFsqc01SB46JvROS7bZs3IO2EmfFsd15uHvIt+Y8vEf7N7fWAU" crossorigin="anonymous">

    <script th:inline="javascript">

        document.querySelector("#frm").addEventListener('submit', function(e) {
            e.preventDefault();

            resetPassword();
        });

        function checkUserName() {
            const data = document.querySelector("input[name=username]").value;

            let isExist = true;
            $.ajax({
                type:"POST"
                , url: "/reset/password/check"
                , data: JSON.stringify(data)
                , dataType : "json"
                , contentType: "application/json"
                , cache: false
                , success: function(data) {
                    let alert = document.querySelector(".alert.alert-danger");

                    if (empty(data)) {
                        alert.innerText = "";
                        alert.style.display = "none";
                        isExist = true;
                        document.querySelector(".btn.float-right.login_btn").removeAttribute("disabled");
                    } else {
                        alert.innerText = data;
                        alert.style.display = "initial";
                        isExist = false;
                        document.querySelector(".btn.float-right.login_btn").setAttribute("disabled", "disabled");
                    }
                }
            });

            return isExist;
        }

        document.querySelector("input[name=username]").addEventListener("keydown", key => {
            if (key.keyCode == 13) {
                resetPassword();
            }
        });

        function resetPassword() {
            const isExist = checkUserName();

            if (!isExist) {
                return;
            }

            const userName = document.querySelector("input[name=username]").value.trim();
            const data = JSON.stringify({ username : userName });

            $.ajax({
                type:"POST"
                , url: "/reset/password"
                , data: data
                , dataType : "json"
                , contentType: "application/json"
                , cache: false
                , success: function(data) {
                    let alert = document.querySelector(".mb-0");
                    alert.innerText = data;

                    document.querySelector(".alert.alert-success").style.display = "block";
                }
            });
        }
    </script>
</html>

```

### 전체 프로젝트 코드

- [https://github.com/jungguji/wordbook](https://github.com/jungguji/wordbook)

* * *

## 참고 사이트

- [https://developer.mozilla.org/ko/docs/Web/HTTP/Status](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)

- [https://developer.mozilla.org/ko/docs/Web/API/Event/preventDefault](https://developer.mozilla.org/ko/docs/Web/API/Event/preventDefault)