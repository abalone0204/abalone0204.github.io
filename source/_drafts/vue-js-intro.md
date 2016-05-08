---
title: Vue.js 到底好在哪？
date: 2016-04-29 23:16:52
tags: [vuejs, front-end, javascript]
---

最近 `Vue.js` [釋出 2.0 的消息](https://zhuanlan.zhihu.com/p/20814761)，

第一次看到 `Vue.js` 我一樣是有點反彈：

「蛤？怎麼這麼像 Angular？」、「居然要用`new`？」

不過試著去摸過一遍之後，我有了一些不同的看法，

這裡把它記錄下來。

<!--more-->

# 前言：先來談談開發環境設定

從 High level 的角度去看，

我想對程式的美感，每個人都是主觀的。

所以我想先來談 dev tools，我想這也是前端生態一個很特別的地方，

就是開發環境設定超他媽麻煩。

包括了 webpack、npm、babel 的設定。

> 我知道還有 grunt、gulp，

> 可是那時，我連程式語言是啥都不知道。

至今仍然有許多人仍然不把 tools 當一回事，

覺得原生的 javascript、css 就很好了，

我們沒必要做那些有的沒的。

這是現在前端「工程」一個蠻悲哀的地方，

未來有機會應該會在寫一篇文章講，如果要在一句話內解釋完我想說的話那大概就是：

「在汽車發明之前，人類都以為馬車就已經夠快了。現在的公路上，是馬車還是汽車呢？」

# vue-cli

沒錯，vue 有一套挺好用的 cli，

讓你不用擔心開發環境的問題。

```bash
$ npm install -g vue-cli

$ vue init webpack my-project

$ cd my-project
$ npm install
$ npm run dev
```

我知道很多人都有自己習慣的套路，不過對於剛要起步的人而言，

這真的是非常非常貼心的事情，

而且 `vue-cli` 也有蠻多選項可以選的，可以參考一下[它的 github](https://github.com/vuejs/vue-cli)。

# 定位

要了解 vue 之前，知道它的定位在哪裡是很重要的，

唯有瞭解這一點你才會明白**哪些是 `vue` 該做、哪些不是**。

用非常粗略地講就是，React 做什麼事情， Vuejs 就做什麼事情。（XD

vue 基本上就是一個 `MVVM` 模式中的 `View Model`。

ViewModel 跟 Model 不一樣，View Model 是 Model of view，

專指做為 View 的 Model，

而一般 MV* 中所指的 Model 則是 domain model。

> 還是太抽象了嗎？

View Model 做的事情就是告訴 View 中的這些東西該綁定（bind）到 Model 中的哪裡，

接下來 View Model 就會自動去幫你處理對應的事情，

沒錯，就是只有數據綁定（Binding）而已，

所以你不該預期他去幫你處理單向資料流⋯⋯等雜七雜八的事。

假如還有興趣的話，最下方有一篇我覺的很棒的參考資料可以看。

# 什麼情況下我不該用 Vuejs

就像所有的 MVVM 一樣，畫面太簡單的話，

使用 vuejs 或是 React 都是一種殺雞用牛刀的行為。

But，人生就是有這個 But，

JavaScript 在搭配 CSS 上簡化了前端開發非常非常多，

> e.q: CSS Module, postcss⋯⋯等好用的配套工具，以及 Component 的設計

假如是有需要長期維護的專案，或是「有可能變複雜」的專案，

我會建議一開始就架好環境直接上 MVVM 了，

與其想到殺雞焉用牛刀，

我反而會反思「畫面很簡單，就算多費一點工，其實也不會花太多力氣」，

但我得到的會是一個更好維護、有個多可能性的產品。
 
而寫過一次就再也不會回頭看他的東西只有兩種可能：

- 你是 Dijkstra，寫一次就完美了

- 你在寫垃圾

我是比較務實一點的人，所以通常都會選擇一開始辛苦一點去保留未來開發的彈性⋯⋯

# ~~Hello world~~ Todo List

Hello world 真的是太沒意思的例子，

我們直接來做 Todo list 吧！

但在這之前，要掌握一些關於 vuejs 的基礎知識。







# 參考資料

- [vue-cli](https://github.com/vuejs/vue-cli)

- [vuejs: instance](http://vuejs.org.cn/guide/instance.html)

- [介面之下：還原真實的 MV* 模式](https://github.com/livoras/blog/issues/11)

