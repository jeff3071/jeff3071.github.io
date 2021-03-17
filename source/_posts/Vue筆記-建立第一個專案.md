---
title: '[Vue筆記]建立第一個專案'
date: 2021-03-03 10:35:34
tags: [Vue, Javascript]
categories: Vue筆記
---
## 相關套件

- node.js
- vue cli

<!-- more -->
## 開始專案

在command line輸入

```
vue init webpack <project name>
```

vue會自動建立一個資料夾，初始化完後移動到資料夾內，並用npm安裝相關套件

```
cd <project name>
npm install
```

最後輸入

```
npm run dev
```

就可以在`localhost:8080`看到初始的畫面了

## 修改頁面

在開發的時候只需要修改`/src`底下的檔案就好，現在的`/src`檔案結構為

```
.
├── App.vue
├── assets
│   └── logo.png
├── components
│   └── HelloWorld.vue
├── main.js
└── router
    └── index.js
```

現在先開啟`/components/HelloWorld.vue`，可以看到`<script>`裡有宣告data的區塊，修改`msg`的值網頁上的文字就會隨著修改。

接著我想要讓使用者輸入值，然後顯示在網頁上，這會用到`v-model`，也就是需要先宣告一個input和變數

```htmlmixed=
<template>
  <div class="hello">
    <h1>{{ msg }}</h1> 
    <input type="text" v-model="hello">
    <h2>{{ hello }}</h2>
  </div>
</template>
```

> template其它的程式碼都刪掉了

接著在`<script>`內宣告hello

```javascript=
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      hello: 'hello'
    }
  }
}
</script>
```

這樣就完成了，修改完網頁會自動更新，不需要重整或重新輸入指令。

## computed屬性

[官方文件](https://cn.vuejs.org/v2/guide/computed.html)

如果想要將輸入得值做計算可以直接

```htmlmixed=
<template>
  <div class="hello">
    <h1>{{ msg }}</h1> 
    <input type="text" v-model="num">
    <h2>{{ num * 100 }}</h2>
  </div>
</template>
```

但如果要做複雜的計算，`computed`可以宣告一個function來處理，方便許多


```javascript=
<template>
  <div class="hello">
    輸入值*100 <input type="number" v-model="num">
    <h2>{{ num * 100 }}</h2>
    <h2>{{ new_num }}</h2>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      num: 0
    }
  },
  computed: {
    new_num: function () {
      return this.num * 100
    }
  }
}
</script>
```

### 利用compute做出簡單的加減

跟前面的例子差不多，但是在`+`的運算下會是字串加減，像是 1+ 1 = 11，所以在做數字運算的時候，需要轉成int

完整程式碼

```javascript=
<template>
  <div class="hello">
    <input type="number" v-model="num">
    +<input type="number" v-model.number="plus">
    -<input type="number" v-model="minus">
    結果： {{ res }}
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      num: 0,
      plus: 0,
      minus: 0
    }
  },
  computed: {
    res () {
      return parseInt(this.num) + parseInt(this.plus) - parseInt(this.minus)
    }
  }
}
</script>
```

結果
![](https://i.imgur.com/1PME0l7.png)

接著會做些小專案來練習