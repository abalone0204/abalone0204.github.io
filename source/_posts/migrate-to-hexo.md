---
title: migrate-to-hexo
date: 2016-04-25 01:17:12
tags: hexo
---

雖然是在同個 repository，而且也是用 github page 來 host，

但這次把 travis CI 加進來以後，改部落格變得方便多了！

這樣就沒有理由阻止自己偷懶不寫文章了吧 XD

<!--more-->

- [hexo](https://hexo.io/)

跟蠻多 static page generators 一樣，

可以跟 github page 做很好的搭配，

只是這一次，把 travis CI 也整進來了。

原因很簡單，個人的部落格，內容是**公開的**，

而 travis CI 對於 open source 的專案則是永久免費的。

實在沒有不用的理由 XD


不過部署的時候，會需要有寫入 repository 的權限，

我參考了以下這篇文章：

- [用 Travis CI 自動部署網站到 GitHub](https://zespia.tw/blog/2015/01/21/continuous-deployment-to-github-with-travis/)

寫的很清楚 XD

我自己是另外 gen 了一個 public key 跟 private key，

所以會需要注意一下命名，

卡了半小時在那邊處理字打錯的白痴錯誤。

基本設定這樣就足以，接下來再來把 ga 和 留言功能加進來了。