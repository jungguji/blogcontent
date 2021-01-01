---
layout: posts
title: 'Ajax에서 SUCCESS가 JSON이면, 전송 할 때도 JSON으로'
date: 2020-04-20 14:33:14
tags:
    - ajax
    - json
---
# 문제
토이 프로젝트 진행 중 문자열 하나만 서버로 넘겨 받아 처리하면 되는 상황이여서 아래 코드 처럼 dataType을 'text'로 설정했다.

```javascript
$.ajax({
	type:"GET"
	, url: "/test"
	, data: data
	, dataType : 'text'
	, cache: false
	, success: function(data) {
		drawGrid(data);
	}
});
```
허나 위 처럼 하니, success에서 받는 data가 json object가 아닌 String type으로 받아지는 문제가 발생했다.

* * *
# 해결

```javascript
$.ajax({
	type:"GET"
	, url: "/test"
	, data: data
	, contentType: "application/json"
	, dataType : 'JSON'
	, cache: false
	, success: function(data) {
		drawGrid(data);
	}
});
```

결국 위 처럼 dataType을 'JSON'으로,
contentType도 'JSON'으로 변경하고
Controller도 json으로 변경하니

success에서 받는 dat도 정상적으로 JSON Object로 넘어왔다.

* * *
# 결론
### 받는 데이터가 json이라면 전송 때 json일 필요가 없어도 json으로 맞춰주자.