---
title: 我所理解的 Dynamo 與實作
tags: [dynamodb, nosql, deep]
---

> 儘管這篇主題是 DynamoDB，但理解 Dynamo 的理論，

> 其實對於 backend 以及分散式運算的想法都能有所助益。

身為一個開發者，我「盡可能」不用自己不了解的東西。

> 理想中當然是不能不求甚解，

> 不過「多聞闕疑」，更接近我自己的態度。

對於剛接觸 Back-end 以及分散式運算的我，

了解 Dynamo 的概念對我而言是相當具有啟發性的。


這篇主要會提到的點是：

- 什麼時候我會需要 NoSQL

- What is DynamoDB？

- Paper: "Dynamo: Amazon's Highly Available Key-value Store"

- NoSQL

- 缺點


假如是一些在官方文件就有的介紹，或是太細節的 API，

我並不會在這裡深入去介紹，

一如往常的，這篇筆記將會偏重在理論及概念上，

因為當能夠深入理解它背後的設計哲學，

才能更知道它的極限在哪裡，

深入去思考也才會更明白自己身為一個 Programmer 的價值在哪裡。

而這裡也不預設需要有什麼知識，

可以碰到不懂的地方再去查，或者是看看補充資料。

> 另外，DynamoDB 並沒有完全照著 Paper 中的 Dynamo 去實作

> 怕標題會誤導了這一點，所以提早在這邊說，

> 但為避免搞混，所以一開始就先把這問題放在一邊。

# Introduction

你可能聽過 DynamoDB，也可能沒聽過，

但聽過、用過，到實際理解過中間又是一大段距離。

最基本的理解是：

> Amazon 的 NoSQL database service。

你可能連用都沒用過它，或者只是用爽爽的不用在乎它背後的理論是什麼，

但你可能不知道 Amazon 的 Dynamo 論文也影響了 Database 及分散式運算。

假如你也懷疑 DynamoDB 到底有什麼能耐的話，

這裡有一篇之前批評 Dynamo 的文章：

[Dynamo: A flawed architecture (jsensarma.com)](https://news.ycombinator.com/item?id=915212)

裡面有提到一些 Dynamo 的缺點，而 Amazon 的 CTO 也有出來回應：

<blockquote class="twitter-tweet" data-lang="zh-tw"><p lang="en" dir="ltr">Darn, someone figured out that Dynamo is a flawed architecture. Luckily its only use is storing hundreds of millions of shopping carts :-)</p>&mdash; Werner Vogels (@Werner) <a href="https://twitter.com/Werner/status/5345892061">2009年11月1日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

嘴砲歸嘴砲，至少我們知道 DyamoDB（雖然它並不等價於 Dynamo Architecture），

的確有實際而且大量的運用。

# Dynamo Paper

> Paper 上面的文字會用的更嚴謹一些，

> 我寫這篇筆記的目的不是為了重新詮釋 paper

> 而是引起更多人的興趣踏出舒適圈去挖掘更深層的知識

quorums

peer to peer database

consistency, conflicts

CAP

- consistency

- Availability

- Partition tolenrance

# Implementation


## Debug

## Create table

- 如何建立 GSI



# References

- [](http://book.mixu.net/distsys/)

- [Amazon DynamoDB](http://docs.aws.amazon.com/zh_cn/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)

- [Dynamo: Amazon's Highly Available Key-value Store](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)

- [Dynamo: A flawed architecture (jsensarma.com)](https://news.ycombinator.com/item?id=915212)

## Video

- [DynamoDB - The Paper That Changed The Database World](https://www.youtube.com/watch?v=-4bS6V1rEb4)