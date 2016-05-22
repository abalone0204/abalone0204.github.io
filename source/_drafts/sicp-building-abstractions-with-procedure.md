---
title: 成為一個 Programmer（一）： 對計算過程的抽象化
tags: [sicp, abstraction, plt, programmer]
---

很多時候，我們會覺得需求變動太快、技術變動太快，

身為一個 Programmer 到底該保有著什麼呢？

有些人會說做事的能力、溝通的能力，這些當然都相當重要，

但我們得面對現實，

不管你的職稱是掛 PM、工程師、程序員、架構師，

最終考驗我們是否能達成目標的以及真正有價值的地方，

其實是對問題抽象化的能力。

這一系列文章就是著重在如何去良好的抽象化思考問題，

並且寫出好的 Program 去解決問題。


這一系列筆記是從 SICP (Structure and Interpretation of Computer Programs) 這本書上，

記下並論述一些比較深刻的想法（因為他是能夠免費被再製的），

所以也很歡迎任何人分享或者討論。

> 這本書是開源的，所以理論上你可以在 [MIT 的網站找到它](https://mitpress.mit.edu/sicp/full-text/book/book.html)

> 這不是一個 SICP 的速讀，更多的是我個人對於「寫程式」這件事的理解

> 當然，裡面也有大量從 SICP 上學到的實用概念

<!--more-->

# Prerequisite

- No，但我並不覺得這是一本簡單的書。

- 不懂 Lisp 甚至不懂寫程式也沒有關係。

- 安裝 Chez Scheme（最近[開源了](https://github.com/cisco/ChezScheme)）

```bash
git clone git@github.com:cisco/ChezScheme.git
cd ChezScheme
./configure
sudo make install
```

## 為什麼用 Lisp

這本書裡的語言是用 Lisp，

確切的說，我們應該是用 Lisp 的方言：Scheme。

Lisp 的語法簡單，並且沒有太多的特例，

所以使用 Lisp 來闡述，能讓我們相對專注在抽象化思考，

而不是「語言特性」上。

另外，許多現在流行的腳本語言（Python, Ruby, JavaScript），

都有許多借鏡於 Lisp 的地方，許多概念都是能夠推導到實際的應用上，

所以這並不是一個打高空取向的筆記，我很重視概念和想法，

但我更重視的是所學的東西，是不是實務上可用的。

# Procedure 程序

程序是什麼？就是我們對於「**數據**」的「**計算過程**」，

下一章節才會提到關於「數據」的抽象化。

這裡讓我們專注在「程序」上面。

說到底，其實我們寫程式，就是在寫下計算過程，

它可能能被重複的執行。

在拆解下來，所有的程序只包含三種元素：

- 基本表達方式（Primitive expression)

    - 像是我們能直接在程序中寫下 `1`，代表的值就會是 `1`

- 抽象的方法(mean of abstraction)

    - 用抽象的符號來代替值

    - e.q: 定義 variable 或是 functions

```bash
> (define x 1)
> x
> 1
```

- 組合的方法(means of combination)

    - 可以用簡單的 Procedure ，組合出更複雜的 Procedure

```bash
> (define )
```