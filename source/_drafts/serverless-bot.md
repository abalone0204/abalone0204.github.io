---
title: 理解 Serverless 架構與實作
tags: [serverless, aws-lambda]
---

Serverless 不代表沒有 server，

我們畢竟還是工程師，不是魔法師，

只是你不用去擔心 server 的 maintain 而已。

這篇文章最後會以 aws lambda 搭配 serverless framework 或是 apex 為例。

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

# 參考資料

- [谈谈AWS Lambda和serverless architecture](https://zhuanlan.zhihu.com/p/20297696)

- [AWS Lambda Dodumentation](https://aws.amazon.com/tw/documentation/lambda/)

- [CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)