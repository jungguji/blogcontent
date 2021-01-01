---
layout: posts
title: Vue 개발 순서 정리
date: 2020-11-10 19:10:35
tags:
    - Vue.js
---

## 서론

Spring boot + Vue를 활용해 토이 프로젝트 진행하고 있는데 헷갈리는 부분을 차후 또 Vue로 개발 할 일이 있을 경우 참고하기 위해 글로 남겨 놓는다.

## 개발 순서

* Vue는 설치되어 있다고 가정한다.

1. root 디렉토리에 vue.config.js 생성하고 필요한 설정 추가

### vue.config.js

```javascript
const path = require("path");

module.exports = {
  assetsDir: "static",
  outputDir: path.resolve(__dirname, "../backend/src/main/resources/static"),
  devServer: {
    proxy: {
      "/": {
        target: "http://localhost:9312",
        ws: true,
        changeOrigin: true
      }
    }
  },
  chainWebpack: config => {
    const svgRule = config.module.rule("svg");
    svgRule.uses.clear();
    svgRule.use("vue-svg-loader").loader("vue-svg-loader");
  }
};
```
2. views/ 에 화면이 될 vue 파일 생성
3. src/ 에 api 폴더 생성
4. api/ 에 화면 별로 필요한 js 파일 생성
   1. 이 js 파일들엔 각 컴포넌트 별로 필요한 api 주소를 호출 할 수 있는 js 코드를 작성한다.
5. api 호출 시 공통으로 들어갈 header를 포함하는 common.js 생성

### common.js

```javascript
import axios from "axios";

const AXIOS = axios.create({
  headers: {
    "Access-Control-Allow-Origin": "*",
    "Content-Type": "application/json; charset = utf-8"
  },
  timeout: 1000
});
```

6. components/ 에 2번에서 만든 화면에 출력 될 component vue 파일 생성
7. router/index.js 에 추가한 vue path 추가
   * rotuer에 추가하지 않으면 화면이 나오지 않음

### router/index.js

```javascript
...
  {
    path: "/main",
    name: "Main",
    component: () => import("../views/Main")
  }
...
```