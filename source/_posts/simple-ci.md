---
title: Travis CI + github page
date: 2016-04-24 21:48:35
tags: CI, Travis CI, github page
---

這週把原本自幹的靜態部落格轉到 hexo 去，

想說辛苦都辛苦了，來整個 CI 也不錯。

這篇主要紀錄一下如何用 Travis-CI + hexo 在 github page 上發布部落格

<!--more-->


# Prerequisite

這裡會拿 [hexo](https://hexo.io/) 來當例子。

因為 hexo 也有自己的 deploy 工具，

估計這種生成靜態頁面的 framework ，

都能相當容易的跟 travis CI 以及 github page 整合起來。

> 唯一要注意的是這裡的 repository 必須是公開的

# Issues

會有兩個主要問題需要解決：

1. themes 是用 submodule 更新的

2. 我們會對 github 上的 repository 有寫入，考量到安全性，我們必須妥善處理 deploy key


# Setup

在專案的根資料夾底下新增 `.travis.yml`

```yml
sudo: false

language: node_js

cache:
  directories:
    - node_modules

notifications:
  email: false

node_js:
  - '4'

before_install:
  - npm i -g npm@^2.0.0

before_script:
  - npm prune

script:
  - npm run deploy

branches:
  only:
    - deploy
```

- 待續

# 參考資料

- [用 Travis CI 自動部署網站到 GitHub](https://zespia.tw/blog/2015/01/21/continuous-deployment-to-github-with-travis/)