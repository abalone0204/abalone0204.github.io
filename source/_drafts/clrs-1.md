---
title: Algorithms - 1 algorithms analysis
tags: [algorithms]
---

面試到底該不該考演算法這個議題吵太久了，

唯一可以確定的是「學習演算法」這件事絕對不會吃虧的。

> 當然，這個前提是如果你有多餘的時間能夠投入的話

而且到了要面試才去刷 leetcode、hacker rank，

那得要有智多星的天份才行，

所以資質平庸如我，還是平常就來複習一下[Introduction to algorithms](https://mitpress.mit.edu/books/introduction-algorithms)這堂課比較實在。

<!--more-->

# Introduction Algorithm

這本書可是大名鼎鼎的 CLRS，

是由四位作者的名字中的字母組合而成。

> 本來叫做 CLR

而且絕對不是 Introduction 等級的書，

除了要求對程式的理解以外，

對數學的要求也尤其嚴格，

但基本上只要修過「機率」，就不會有太大的問題。

不過得放在心裡的是，這不是一門「數學課」，

而是一門「電腦工程」與「數學理論」兼具的課程。

要有數學的嚴謹以及工程的直覺。

# How does this series work

基本上就是照著 [course site](http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-046j-introduction-to-algorithms-sma-5503-fall-2005/)

上面給的資源走。

# Algorithms Analysis

- 簡言之我們估的不是「實際執行的時間」，而是隨著 input size 增加，時間增長的「速率」

# Asymptotic notation

- Big-o notation

- e.q: 如果是 `n^3+n^2+n+`，那就用 `O(n^3)`來代表

    - 如果 n 趨近於無限，我們可以可以找到一個很大的 `n0` 確定 `O(n^2)`絕對比 `O(n^3)` 快，不論其他參數如何

- 不過從工程的角度看來，有時候這個 `n0` 大到電腦無法處理時，我們會選擇一個相對低速的算法

# Insertion sort（插入排序法）

- 什麼是 insertion sort?

    - [wiki](https://zh.wikipedia.org/wiki/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)

- worst case: reversed sorted

- T(n) = 1+2+3...+(n-1) => O(n^2)

- `O` 是 weak notation，不能直接拿來約分


# Merged sort


