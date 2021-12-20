title: vue项目中使用vuex存储变量
date: '2021-05-15 11:37:58'
updated: '2021-05-15 11:37:58'
tags: [vue]
permalink: /articles/2021/05/15/1621049877913.html
---
# 前记

因为项目前端需要访问后端的api, 在写axios的时候发现有写了几个地方都是同一个api连接, 于是就想写入"配置文件"中以便直接读取和解耦. 在vue的官方文档中驰骋了一会, 发现因为版本不一样, 不知道怎么引入项目, 当场裂开. 好在后来还是找到了.
(主要怕自己下次忘了所以记一下)
# 环境

`vue3`
`@vue/cli 4.5.12`

# 简单使用

1. 创建store文件夹位于`project/src/`, 文件夹内有一个index.js文件
2. 在state中写入要存储的变量

```javascript
store/index.js
import { createStore } from "vuex";

export default createStore({
  state: {
    food:"鸡扒饭",  
    anotherFood:"盖浇饭"
  },
  mutations: {},
  actions: {},
  modules: {},
});
```

3. 在main.js中使用store

因为**版本的差异** , 我项目中使用的是`createApp.use(store)`

```javascript
// main.js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";
import store from "./store";

createApp(App).use(store).use(router).use(store).mount("#app");
```

而网上有一些版本则是`new Vue()`

```javascript
// 其他版本
// 省略import
new Vue({
    el: '#app',
    store,
    ... // 省略其他内容
})
```

4. 在component中使用$store.state.var获取变量

```html
<template>
    <h1>{{ $store.state.food }}</h1>
    <h1>{{ $store.state.anotherFood }}</h1>
</template>
```

在<scrpit></script>中也能使用, 只需要在前面加上this.即可, 例如 `console.log(this.$store.state.food)`

# 其他使用方法

因为官方文档中只是没有说明要如何将vuex引入到项目中, 因此也就不记录其他例如`mutations`的用法的, 感兴趣的可以自行查看官方文档.

# 参考

[vuex使用demo - 简书](https://www.jianshu.com/p/b5dace21e63f)
[vuex官方文档](https://vuex.vuejs.org/zh/guide/)