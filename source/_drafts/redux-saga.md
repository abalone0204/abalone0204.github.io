---
title: `redux-saga`
tags: [redux, saga, redux-saga, generator]
---

`redux-saga`到底是在解決什麼問題？

讓我們的非同步 action 能夠更好被開發、維護、測試。

> 這裡來舉個例子：我們要登入

```
送出登入 request =>
畫面進入 loading 畫面 =>
if (登入成功) {
    取得並把 token 快取起來 => 
    redirect 到首頁 => 
    done
} else {
    顯示錯誤訊息在首頁上
}
```

你會怎樣去設計這個資料流呢？

畫面要有什麼 state 呢？

這個流程看似簡單，但要處理的乾淨、又好測試，

是不是事情就沒有那麼直覺了？

這裡看起來好像很抽象，但瞭解過後，

`redux-saga` 其實並不是太特別的概念。

我不認為 `redux-saga` 的只是拿來取代 `redux-thunk`的工具，

重要的應該是 saga 這個 pattern 背後的概念，

`redux-saga` 將會給你新的方式去思考前端資料流。

我認為如果有出現以下幾個現象，

那 redux-saga 值得一試：

- 學會 generator function 卻無處可應用

- 處理非同步的 action 時，總覺得哪裡怪怪的

- 純粹好奇 `redux-saga`能幫助你什麼

<!--more-->

# Intro

有些人會說 `redux-saga` 的學習曲線比較陡峭，

其實並不盡然。

會覺得 `redux-saga` 太過困難，

通常就是因為一次就想直接學會、並應用，

忽略有些預先知識必須要一步一步學習，

而且有些情況，必須拉高一點視角會比較好看清楚，

從概念的角度去看，而不是只關注在前端的實作。

> 很多人會認為 Database 是後端的事情
> 實際上我的後端觀念有許多需要加強的地方
> 還稱不上是 fullstack，
> 中間少了一大段，就只是個 fuck 而已

我認為這裡只有三件事情要掌握

- 什麼是 saga？

- saga 跟前端開發有什麼關係？

- redux-saga 的基礎用法

# 什麼是 Saga

要學一個東西，把名詞搞懂是很重要的。

像 router 就是個很直覺又常見的名詞，

saga 是什麼呢？

`redux-saga` 有提供一些資源供參考，

包括了最原始提出 saga 這個 pattern 的[論文](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)。

一共 11 頁，不過扣掉 acknowledgment 跟 References ，

就只有 9 頁半啦！

不過論文中是從 database 的角度看，

另一個影片更進階一點，是從應用在分散式系統的角度去解釋。

saga 其實是個很簡單的概念，

要應用它也並不困難，

這篇論文在 DBMS 上實作的原因，

主要只是要闡明如何實做一個簡潔、有效率的 sagas，

所以不要擔心接下來講的例子看起來跟 redux 或前端開發沒有關係，

稍後會提到要怎樣在前端開發中應用 saga 這個 pattern。

所以看個幾分鐘之後，腦袋裡會冒出許多的問號：「所以 saga 是⋯⋯？」。

這裡我試著用最簡單的語言解釋 saga 是什麼。

Saga，就是個滿足特殊條件的 LLT(Long lived transaction)。

> 待會會說是什麼特殊條件。

> 如果你不知道什麼是 Transaction：

> 是 Database 上常會用到（但不僅止侷限於 Database）的名詞，

> 即是「交易」。

> 「交易」聽起來很抽象，

> 其實他要敘述的就是銀貨兩訖後，

> 一個交易才算是完成，

> 假如銀貨不兩訖的話，那要退回最一開始的時候，

> 買賣雙方的狀態不會有任何改變。

## Long lived transaction (LLT)有什麼問題：

Long lived transaction 是什麼呢？

像是記錄銀行一整個月的狀態，

如果中間有任何一筆資料丟失了，

那就算是「交易(transaction)」失敗，

要退回一開始的狀態。

聽起來，似乎是很糟糕的概念對吧？

為了實現 transaction，我們通常會把正在 transaction 中的 object lock 住，

讓其他人沒辦法更動它。

（維持資料的 consistency）

所以這麼長時間的 transaction，

聽起來會造成很長時間的 delay、外加很高的失敗率(failure rate）。

## Saga 是一種特殊的 LLT

這裡拿演唱會的門票定位來舉例，

> `saga`: LLT
that can be broken up into a collection of subtransactions
that can be iterleaved in any way with other transactlons 

