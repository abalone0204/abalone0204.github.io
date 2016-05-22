---
title: 理解 Serverless 架構與實作
tags: [serverless, aws-lambda]
---

Serverless 不代表沒有 server，

我們畢竟還是工程師，不是魔法師，

只是你不用去擔心 server 的 maintain 而已。

這篇文章最後會以 aws lambda 搭配 serverless framework 或是 apex 為例。

身為一個軟體工程師，

我們大部分都在追求低耦合（decoupling）的程式碼，

serverless 以 function 為單位，

看起來好像達到了這件事，實際上卻是像壽司一樣。

(圖：如何做壽司) 

了解如何活用而不搞砸就更考驗我們設計系統的技巧了。

> 幹他媽我寫到一半也差點就要放棄了


<!--more-->

# Catalogue

# Intro

跟以往一樣，

會先從為什麼需要學習 serverless 的架構，

以及什麼人需要這種架構，

最後在淺淺談一下 serverless 的哲學，

接著，才是實作。

> 我並不會直接開始就啪啪啪的將工具的使用流程寫上來，

> 要這樣的話，其實沒必要寫下文字紀錄思考的脈絡。

> 有一本關於電腦科學的好書裡面關注包含三個面向：

> - 人的大腦

> - 程式碼

> - 電腦（硬體）

> 這本好書叫做 SICP(Structure and Interpretation of Computer Programs)

## 常見的問題

- 資料的持久性

- Log

我知道還有很多其他問題，但基本上只要能符合第一點，

再加上本來能執行的 function，你基本上就能完成一個 application

## 限制

硬體的限制：

- 有限的記憶體

- timeout

軟體的限制：

- 語言，目前支援 nodejs, python, Java

    - 馬的 我還以為會有 Haskell

- 要學習使用 AWS 的 service

    - 至少一定要知道 API gateway 以及 lambda

- 思維模式要改變，專注在 function 上

    - monotholic, microservices,nonoservice, mixin

    - 要寫很多 config 就是在幫我們省掉維護機器的那份工

    - 最難的其實不是寫，而是開頭的學習

## 好處

- 蠻明顯的，不需要自己管機器，以及近乎無限 scale-out

- 如果只是自己要使用或是小型專案，基本上都會落在 free tier 區間
    -  pricing 待查



## 理念

Event driven

- 完成事件流，基本架構也就完成了

## 陷阱

- 失控的 decoupling => 
    - 這正是為什麼會需要 framework 的原因 
    - 學習好的架構（可能不是最好，但比自己硬幹來得好）

# Implementation

- 打 request 進去有辦法存進去 db

- get 資料出來


## AWS lambda

分成兩個 component:

- Lambda function

- event source


handler:

>As part of the configuration, you also specify a handler (that is, a method/function in your code) where AWS Lambda can begin executing your code. AWS Lambda provides event data as input to this handler, which processes the event.



## Setup

建立一個 admin 的 user。

記下 access key id 以及 secret access key。

> 其實可以下載下來 

再來就是照著官網的安裝好 `serverless-framework`，

沒錯，用 npm 即可完成。

```
npm install serverless -g
serverless project create
```

再來就是輸入 project 的名稱以及 stage，

假如你是第一次創建的話，會需要幫你建立一個 aws profile，

輸入剛剛的 id 和 secret 就可以搞定。

```
serverless function create functions/function1
```

選擇 `create endpoint`。

> 這裡完全可以直接在專案的根目錄直接 create function1

> e.q: `serverless function create function1`

> 不過把相同的 services 放在同一個資料夾底下會是比較好的做法

接著就會看到生成好的專案，

裡面有蠻多看不太懂的東西，不過現在先不用在乎。

## Deploy

好的服務在剛創建的時候，就要先部署上去，

不要等到開發一大包了，才開始準備部署的東西。

> 真的 Q_Q

```bash
serverless dash deploy
```

再來把 endpoint 和 function1 都選擇（會變色），

最後在部署上去。

會得到一個網址，就是呼叫這個 function 的網址，

假如你什麼都沒改的話，應該會看到這行字

```json
{"message":"Go Serverless! Your Lambda function executed successfully!"}
```

## Project Structure

這裡其實蠻想跳過的，不過值得一提的是我們建立這個 project 的架構，

適合部署到 AWS 上的原因就是因為他是用 CloudFormation 生成的，

底下縮寫的 `s` 代表的就是 severless，而 `cf`就是指 CloudFormation 啦！

對於奇奇怪怪的縮寫，就是沒辦法假裝不在意，畢竟查一下也花不了幾秒鐘吧？

> 這裡簡介一下 CloudFormation ，儘管我們只是透過 serverless 來使用它。

> 簡單的說它能生成一個描述我們要使用 AWS 上哪些資源的 json 檔案，

> 類似 `npm` 的 `package.json`，

> 只是變成去描述我們用了哪些 AWS 的服務，以及要如何用而已。


```
├── _meta // (.gitignored) 就是個存 meta data 的地方（config 之類的
├── admin.env // (.gitignored)剛剛 create function 時的 AWS Profiles
├── functions
│   └── function1
│       ├── event.json
│       ├── handler.js
│       └── s-function.json
├── package.json // 就是 npm 的那個
├── s-project.json // serverless 的套件管理
└── s-resources-cf.json // 就是上述講到 CloudFormation 的描述檔
```

這裡略過了一整串，因為我想把它特別拿出來說，

畢竟 function 就是 serverless 的主體。

```
└── functions
    └── function1
        ├── event.json
        ├── handler.js
        └── s-function.json
```

放在 functions 資料夾底下並不是強制的規定，

只是 serverless 認為這是 best practice 而已。

## Function

- 每個 function 可以有許多個 endpoint

- 每個 endpoint 可以有許多個 method( GET, POST...)

- Handler 則是 aws lambda 執行的進入點

來看一下 handler.js

```js
module.exports.handler = function(event, context, cb) {
    // empty
}
```

第一個參數 `event` 就是我們前面提到的 source event，

第二個 `context` 則是 lambda function 在運行時跟整個環境互動，以及環境的訊息。

> 這裡有點抽象 等到後來例子在講解會更清楚一點


- logging： CloudWatch

## Programming model: Nodejs

我們一直在談 lambda function，

實際上我們運行的 function 就是長下面這個樣子，

在開始討論其他配置，和 aws 要怎麼運行到這裡之前，

先搞清楚到底在談論什麼東西：

```js
function(event, context)
```

> 可以有第三個參數 cabllback，
> 不過其實只要這兩項就可以運作的很好了，
> 而且 callback 實在不是一個好事

## event

source event，可以是 push 或 pull model。

## context

`context` 參數是一個 object，

裡面包含了當前 lambda 運行環境的訊息，

以及一些 method。

有三個 methods 是一定要知道的：

> 這裡的參數是可選的，我們可以只讓 function 做事，
> 沒有一定要強制回傳結果。

- `context.succeed(Object result)`
    
    - 可以在執行成功時回傳東西： `context.succeed(someObject)`

    - 注意這裡的 `result` 必須要能夠被 JSON.stringifyu 轉成字串

- `context.fail(Error error)` 

    - 在失敗時回傳東西

- `context.done(Error error, Object result)`

    - 這個就有點奇葩了，有了成功和失敗為什麼還要存在個 done 呢？


再來是可以看到目前執行剩餘時間：

`context.getRemainingTimeInMillis()`

這裡所謂的看到當然是指在 function 執行時我們能利用啦！

不過要注意的是如果一定歸零，AWS lambda 就會強制終止我們的 lambda function 了。

## handler.js

前面有提到過這裡就是 aws 運行的進入點，

要在 `s-function.json` 裡面設定，

這裡看到我們只在 `handler` 那個屬性打上 : `handler.handler`，

這有兩件事情值得注意：

- 對應執行的就是 `handler.js` 這個 module 底下的 `handler`


```js
// in handler.js
module.exports.handler = function(event, context) {
    // This be implemented
}

```

第二件事就是這個 hanlder 屬性還隱含著我們目前能作用的 scope，

假如我們是：`function1/handler.handler`，

就把上層的 parent folder 給包含進去，

所以他就吃得到我們在根目錄安裝的 npm 套件。

> 比如說你安裝了 react，那你就可以：
> `require('react')`

# 參考資料

- [谈谈AWS Lambda和serverless architecture](https://zhuanlan.zhihu.com/p/20297696)

- [AWS Lambda Dodumentation](https://aws.amazon.com/tw/documentation/lambda/)

    - [Programming model: Nodejs](http://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-context.html)

- [CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)