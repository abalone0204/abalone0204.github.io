---
title: 解析 Sagas 
tags: [sagas, paper]
---

在電腦科學這個廣大的領域裡面，

Library 層出不窮，

背後抽象的概念學會才是真正有價值並且能夠不被輕易取代的，

也因為這樣，促使我去讀了 sagas 原始論文。

> 我是個剛在前端學習的工程師
> 動機只是為了要運用 redux-saga

畢竟論文的用字和語言都比較嚴謹一點，

這裡希望能用較為淺顯的文字，

跟大家說明這個 pattern 的概念以及實作。

<!--more-->

# What is saga?

**Saga**，就是個滿足特殊條件的 **LLT(Long lived transaction)**。

## 什麼是 LLT

是個長時間的 transaction

> 如果你不知道什麼是 Transaction：

> 是 Database 上常會用到（但不僅止侷限於 Database）的名詞，

> 即是「交易」。

> 「交易」聽起來很抽象，

> 其實他要敘述的就是銀貨兩訖後，

> 一個交易才算是完成，

> 假如銀貨不兩訖的話，那要退回最一開始的時候，

> 買賣雙方的狀態會退回交易前的狀態，不會有任何改變。

而 LLT 就是一個長時間的 transaction，

就算沒有受到其他影響，

整個完成可能也需要數小時或數天。

## LLT 有什麼問題

LLT 可能會有一系列的動作(atomic actions)需要被完成，

這帶來底下兩個問題：

- 高失敗率

- 長時間的 lock

為了實現 transaction，我們通常會把正在 transaction 中的 object lock 住，

讓其他人沒辦法更動它。

（維持資料的 consistency）

所以這麼長時間的 transaction，

會造成兩個問題：

- 較高的失敗率

- dead lock 造成的長時間 delay

為解決這個問題，

我們這裡可以假設一個 LLT：`T`

可以被拆成許多相互獨立的 subtransaction的集合:
`t_1`~`t_n`。

但如果我們不會希望`t_1`~`t_n`分別被送進 DB 並且記錄下來，

因為假如有一個失敗的話，

那 `T` 就不算是完成的 transaction。

儘管如此，這樣做也比一般的 transaction 帶來了一些彈性，

我們可以隨意的插入 subtransaction。

有了這個大方向以後，

接著就來解釋 saga 運用什麼樣的設計方式來解決這些問題。

## Saga

第一件要注意到的事就是 saga 仍然是個 LLT。

> `saga`: LLT
that can be broken up into a collection of subtransactions
that can be iterleaved in any way with other transactlons 

作為一個 LLT，

假如任何一個 saga 中的 subtransaction: `t_i` 單獨執行了，

我們應該要有一個 compensating transaction `c_i` 可以將它 undo。

這裡的 compensating 是從語法觀點來看，

而不是說系統的狀態要回到 `t_i`執行的那個時間點。

這一點非常重要！魔鬼藏在細節裡就是這個意思。

> 你可能會覺得這兩件事不是差不多嗎？

> 舉個例子：

> 如果有個 LLT : `T` 是要記住所有買江蕙票的座位數，

> 底下每個訂票都是一個 subtransaction: `t` 。

> 假設 `t_i` 要被買票的人取消，

> 我們執行 `c_i`時，

> 只是把買的座位數從 database 裡面減掉

> 而不是讓 database 回到 `t_i`發生前的時間點




# 參考資料

- [Sagas](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)

