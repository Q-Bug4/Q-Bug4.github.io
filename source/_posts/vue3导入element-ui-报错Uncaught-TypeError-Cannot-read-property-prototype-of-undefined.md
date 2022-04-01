---
title: >-
  vue3导入element-ui 报错Uncaught TypeError: Cannot read property 'prototype' of
  undefined.
date: 2022-04-01 19:53:54
tags:
---
## 环境
```bash
vue --version
@vue/cli 5.0.4
vue 3
```
## 问题
今天使用vue3的时候按照某教程按照如下方式导入`element-ui`：
```js
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"
```

结果`npm run serve`后页面空白，打开Console一看报错了：`Uncaught TypeError: Cannot read property 'prototype' of undefined.`

## 解决
搜索了一下发现`element-ui`是给vue2.x使用的，我们使用vue3需要导入`element-plus`包，通过需要按照[官方文档](https://element-plus.org/zh-CN/guide/quickstart.html#%E5%AE%8C%E6%95%B4%E5%BC%95%E5%85%A5)来进行导入。

官网贴出来的具体导入代码如下：
```js
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)
app.mount('#app')
```

