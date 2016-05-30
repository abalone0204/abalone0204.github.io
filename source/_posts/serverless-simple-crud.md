---
title: 淺析 serverless 架構與實作
tags:
  - serverless
  - aws-lambda
  - dynamodb
date: 2016-05-22 03:32:31
---


Serverless ，不是沒有 server，而是不用去擔心維護 server 這件事，

不管是在部署還是開發，都是以一個個 function 為單位，

這帶來了程式碼上的高度 decoupling，但同時也因為過大的彈性，

常常搞的我們無所適從，就像這張圖一樣：

![howto](http://i.imgur.com/lP3wcnh.jpg)

serverless 更考驗著我們對系統設計的思維，

這是一篇非常粗淺的文章，

目的在帶領對 serverless 有興趣的人無痛的入門，

不管是在概念上，還是在實務的使用上。

> 假如你是懶得看文章的人，可以直接到我的 [github repo](https://github.com/abalone0204/serverless-demo-with-dynamodb-node) 上面看

> 有哪裡寫錯的話可以提個 issue，覺得讚讚讚的話也可以給星星以茲鼓勵。


<!--more-->

試想當你是一個單槍匹馬的開發者時，你絕對會希望能真正專心在開發，

而不是一天到晚擔心機器有沒有死掉，或者配置環境就花了大半時間。

我只是一個前端工程師，對於後端的知識甚是淺薄，

serverless 對我而言是個很合理的選擇，

但這不代表我不在乎任何後端的專業性，

更不代表著後端工程師使用 serverless 架構就是代表實力不夠。

相反的，我認為後端工程師如果能從管理機器中解放，

設計出更好的 serverless 架構以及更專注在程式本身的邏輯上，

那從 serverless 上能獲得的增益一定也是相當驚人的。

> 看著我們虛擬化的趨勢 => VM => Container => Docker 的興起

> 儘管做法略有不同，但方向是一致的，

> 都是想讓程式開發者更能專注在程式本身，而不是管理機器上

> 話說回來，前端後端的分界點一直都是個有爭議的問題，

> 不過就不在這裡去討論了


這篇會需要用到數個 aws 的服務，不過為了讓事情更單純，

我只會用到 IAM, DynamoDB, API Gateway, CloudWatch 以及 Lambda，

都不熟悉這些也沒有關係，因為我在寫完這一段之前， 

也只是大略的把文件掃過去，也不用擔心縮寫令人看不懂，

因為我最討厭的就是這種縮來縮去的東西，

所以接下來都會在提到的地方解釋我們正在處理的是什麼。

> 以往都是直接用 EC2 開一台機器，

> 要用什麼直接當自己家的在上面裝就是了。

> （當然可以學一些東西自動化這流程： chef，不過這不是這篇的重點）

# Catalogue

- [Introduction](#Introduction)

    - [優點](#優點)

    - [限制與風險](#限制與風險)

    - [Why serverless framework](#Why-serverless-framework)

- [Setup 開發環境的建置](#Setup-開發環境的建置)

    - [為你的 api 建立一個「role」](#為你的-api-建立一個「role」)

    - [Create Project](#Create-Project)

    - [Create First function](#Create-First-function)

    - [Deployment](#Deployment)

- [Abstraction](#Abstraction)
    
    - [Overview](#Overview)

    - [Source event](#Source event)

    - [Context](#Context)

    - [`handler.js`](#handler-js)


- [Implementation: Simple RESTful api: Simple RESTful api](#Implementation-Simple-RESTful-api)

    - [Why](#Why)

    - [Log](#Log)

    - [Create an item](#Create-an-item)

    - [Read an item](#Read-an-item)

    - [Update an item](#Update-an-item)

    - [Delete an item](#Delete-an-item)

    - [List items](#List-items)

- [Conclusion](#Conclusion)

- [References](#References)


# Introduction

這篇會著重在比較抽象化的概念上，

而不是去針對特定的功能作 serverless 的實現，

> 但不要誤會了，後面還是有一個簡易 restful api 的實作

我認為能掌握以下幾個點，才是針對特定功能實現的基礎：

- Project 的架構

    - 對於設計一套 serverless architecture 的抽象概念

    - 各個功能與 api 間對應的關係

- 資料的處理 
    
    - 要能永久被儲存
    
    - CRUD 操作
    
    - Schedule：定時或是 routine 的去做一些事情(這一篇文章裡面不會提到)

- 部署

    - 有新功能時我們要能夠部署上去

- Log

    - 不然你 debug 是要通靈嗎

至於使用的語言會是 nodejs。

## 優點

- 不需要自己管機器，以及近乎無限能力的 scale-out（你的財力夠的話）

- 相對便宜。因為我們是有執行 function 才收費

    - 如果只是自己要使用或是小型專案，基本上都會落在 free tier 區間
    
- 高度的解耦及靈活的配置

    - 不管你是想要製作 nano service 還是 micro service 你都能靈活地去組合

有人說過，當你手上只有錘子時，那你看到的所有東西都會是釘子。

不過對於 `function` 這麼 general purpose 的東西來說，

它的確能拿來解決一切計算相關的問題，端看你組合的方式對不對而已。

## 限制與風險

講了這麼多好處，現在當然要來講它的限制。

- 有限的記憶體

- timeout

  - 目前最多只能運算 300 秒，就會被強制結束掉

- 高度的解耦

    - 這看起來是好處，但必須要用跟以前不一樣的想法來設計程式，因為我們每次 function 運行完之後，就會把所有資源釋放出去

- Latency

  - 因為我們是需要計算時，才會去要資源來運算，每次都算是一個 cold start，所以對 latency 完全無法容忍的服務，可能不適合。

  - 實際上透過 schedule 可以一定程度的解決這問題

- 風險
    
    - Scale-out
    
        - 坦白說，如果是考慮到有沒有辦法 scale-out，那我想大部分情形，aws 都是沒問題的

    - API 更換
        - 因為我們以 function 為單位的高解耦，所以更換 API，不是一個讓人全面崩潰的風險

    - **服務被停用**

        - 我說一個字大家就懂了：Parse

        - 當事情走到這一步的時候，基本上就沒啥救了，這就是我們冒著最大的風險

        - 但就如同前面所言，我認為 serverless 是未來大勢所趨，也許不會所有的 project 都如此，不過大多數的中小型專案都會轉向朝這一架構邁進。


## Why serverless framework

- 過度的自由，失控的 decoupling

    - 框架給了我們更好結構化 project 的方式

- Config 的設置以及部署 function 簡化

- 文件和 plugins

- 社群或公司支持
    
    - Serverless 的官網上有說到，現在是由一群工程師全職在維護這個 framework

    - gitter 上問問題也幾乎馬上就能得到回答

- Apex?

    - TJ 的產品，目前還在觀望中，但 serverless 看起來相對較穩定、成熟

    - 不過光是 TJ 這個名字，就很值得一試

    - 就像我前面說的，因為高度解耦的關係，其實要遷移過來「理論上」不是太難的事

# Setup 開發環境的建置

我不認為一個環境的建置，是在把東西裝一裝之後就結束了，

因為東西裝一裝之後，通常後續只會有更多的問題，

而且一個 project 本來就需要在一開始就做好 deploy 的準備了。

> 不部署的話幹嘛要用 aws 啊？囧

完整一點的 setup 應該要包含了從 建置基本設定 => 部署 

才算是真的結束，

所以這一小節會從配置到部署都走過一次。

> AWS 的介面可能會因為時間的關係，與下方略有不同，

> 但估計變動不會太大，知道要使用什麼功能比較重要，

> 故我不會把操作介面的圖片放上來。

## 為你的 api 建立一個「role」

- 跟以往一樣，我認為建環境是最困難的部分

- 首先要建一個 `IAM` role

> IAM(Identity and Access Management)

> `IAM` 的功用就是讓你能夠管理使用者對於服務和資源所擁有的「權限」

> 可以針對不同的使用者，制定不同的角色，

> 舉例來說，如果你今天的 api 只想讓 user 從 s3 的 bucket 裡面讀一些靜態資源

> 你就不會想要讓他擁有 access DynamoDB 的權限，懂？

> **IAM 是免費的**。

到 aws 選取 services，在拉下來一狗票的服務中，

選擇 `IAM`。

建立一個新的 User，名字就輸入：`serverless-admin`。

建立好之後，

把拿到的 `Access Key Id` 跟 `Secret Access Key` 給記下來，

待會會用到。

接著選擇剛剛建立的那個 user：`serverlss-admin`，

在 permissions 的地方加上新的 policy，

這裡 aws 相當貼心的提供我們超大一坨的 policies 可供選擇，

為了方便，我們直接選擇 `AdministratorAccess`。

> 當在 production 環境時，這樣處理 permissions 不會是一個好主意 XD

> 坦白說我覺得 permissions 會是一個令人頭痛的點

## Create Project

我們選擇了 `serverless-framework`這一套 serverless framework。

```
npm i -g serverless
serverless project create
```

會要你輸入名字以及剛剛的 access key id 跟 secret access key。

接著還要選擇你想要你的 project 運行服務在的地區。

再來稍後三分鐘之後， project 就會建好了。

> 會生成一大堆東西，下面列出簡易版的解釋，

> 看不懂也沒關係，之後在實作中就會碰到很多次了：

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

## Create First function

先讓我們 focus 在 `function` 上，這些 config 真的都可以先放著沒關係。

> 這不代表他們不重要，只是晚點再回來看他們是在做什麼

> 如果你真的現在就等不及，也可以到 serverless 的官方文件看

> [Project structure](http://docs.serverless.com/docs/project-structure)

```
serverless function create functions/posts
```

選擇 nodejs => Create Endpoint

接著就可以看到多了一個 `functions` 資料夾，

並且裡面跟著一個 `posts` 以及一些東西了。

一樣我們只要知道自己現在建立了一些基礎建設，稍後再來回頭看這是什麼。

## Deployment

```
serverless dash deploy
```

```
function - posts
endpoint - posts - GET
```

這兩個都記得要選才會把東西部署上去 aws-lambda。

選擇 deploy 之後稍待幾秒鐘，就可以看到回傳一個網址給你。

這就是能夠執行我們剛剛部屬上去的 `posts` 的地方。

如果你沒做任何更改，點進去後應該能看到

```
{"message": "Go Serverless! Your Lambda function executed successfully!"}
```

到這裡為止，我們才能不心虛的說：環境建完，可以繼續了。

# Abstraction

## Overview

前面一直說到 serverless 架構是以 function 為單位去部署和開發，

現在來對「lambda function」有個具體的抽象概念。（欸？

先來個大略的概觀，你可以跟剛剛 create 的 project 對照著看：

- 每個 function 可以有許多個 endpoint（進入點）

- 每個 endpoint 可以有許多個 method( GET, POST...)

- Handler 則是 aws lambda 執行的進入點(就是 `handler.js`)

來看一下 handler.js

```js
module.exports.handler = function(event, context, cb) {
    // empty
}
```

實際上我們運行的 function 就是長下面這個樣子，

在開始討論其他配置，和 aws 要怎麼運行到這裡之前，

先搞清楚到底在談論什麼東西：

```js
function(event, context)
```

> 可以有第三個參數 cabllback，

> 不過其實只要這兩項就可以運作的很好了，

> 而且 callback 實在不是一個好事

## Source event

source event，可以是 push 或 pull model。

假設 S3 上面資料新增，lambda function 會接收到 event 去做事情，

那這就是一個 push model。

假設今天是 lamda function 去掃了一遍 DynamoDB ，

發現有事情要根據上面的資料去做，

這就是一個 pull model。

而 source event 也可以很單純的來自 http request。

## Context

`context` 是一個 object，

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

    - 如果 error 不為 null，這次的 lamda function 就會被認定為執行失敗


再來是可以看到目前執行剩餘時間：

`context.getRemainingTimeInMillis()`

這裡所謂的看到當然是指在 function 執行時我們能利用啦！

不過要注意的是如果歸零，

AWS lambda 就會強制終止我們的 lambda function 了。

## `handler.js`

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

理解到這樣的程度，就已經足夠進行下去了，

直接來實作吧！

# Implementation: Simple RESTful api

直接看文件時，總會有種霧裡看花的感覺，

不過等到實際開始做之後，你會發現其實概念只要 mapping 過去，

並沒有想像中的困難。

> 這個是完成後的 [github repo](https://github.com/abalone0204/serverless-demo-with-dynamodb-node)，

> 如果你中途發現有什麼錯誤的話，可以在上面查看是否有哪裡不一樣。


## Why

底下會包含基本的 CRUD 以及 list，

大多數的應用程式都不脫這五種操作，

就算需要更特殊的操作，

也總是要熟悉這些基礎後才能繼續前進，

包含著如何儲存資料以及 debug 的概念。

至於資料夾的結構或是 workflow 的順序，

你都可以依照個人的喜好去調整，不一定要照我寫的走。


## Log 

- 沒錯，我們先來看看要怎麼找出錯誤，從犯錯中學習，是新手成長最快的方式

- 來修改一下`functions/posts/hanlder.js`

`context` 和 `event` 是我們在 lambda 中要好好處理的東西沒錯，

不過這裡先專注在出 bug 時要怎麼解決：

```js
'use strict'

console.log('Loading function')

function display(object) {
    return JSON.stringify(object, null, 2)
}

module.exports.handler = (event, context) => {
  console.log('Event: ', display(event))
  console.log('Context: ', display(context))
  context.succedd({
    message: 'ok, it works'
  })
}
```

> 這裡的程式碼有個明顯的錯誤，待會我們會除錯並且學習如何看 log

稍做一些更改之後我們就可以再次部署了：

```
serverless dash deploy
```

再到剛剛的網址，會發現出現錯誤了！

幸好這裡加上了許多 `console.log`，

假如你曾經寫過 JavaScript 對這樣的除錯技巧一定不陌生，

但，這裡的 log 不會在 console 印出來，會到哪裡呢？

這裡就要使用 aws 上的另個服務：CloudWatch 了。

到 services 點 CloudWatch，選取 logs，

就會看到這裡有個 log groups 就是我們剛剛建立的 functions。

選進去後會很神奇地發現我們之前 call 的紀錄都在這裡。

在 log 中我們可以看到：

```
...(一些日期和系統資訊) TypeError: context.succedd is not a function at module.exports.handler (/const/task/handler.js:12:11)
```

我們出了一個 typo 的錯誤，改正過來以後就成功啦！

```js
context.succeed({
    message: 'ok, it works'
})
```

## Create an item 


要存資料庫前，必須先在 `DynamoDB` 建一張 Table。

> DynamoDB 是一個 no sql 的資料庫

> 為了 scale-out ，它在使用上有一些限制，

> 但在這個簡單的示例中，並不會需要考量到這些，

> 假如有興趣深入的話，可以看補充資料的地方

> [解析 DynamoDB](http://history.programmer.com.cn/11081/)

- 到 aws 上選擇 `DynamoDB` 。

- Create table

- table name 輸入 `posts`

- primary key 名稱設定為 `id`

- 下面的 default setting 取消勾選，然後將 Read capacity units 以及 Write capacity units 都調成 1

- 我們就有一個很陽春的 table 了

接著是在 `handler` 裡面的更動，

首先要安裝兩個 package

```
npm i -S dynamodb-doc node-uuid
```

前面有說過 lambda function 其實就是根據 source event，

去執行對應的動作：


```js
const DOC = require('dynamodb-doc')
const dynamo = new DOC.DynamoDB()

module.exports.handler = (event, context) => {
    console.log('Event: ', display(event))
    console.log('Context: ', display(context))
    
    const operation = event.operation

    if (event.tableName) {
        event.payload.TableName = event.tableName
    }

    switch (operation) {
        case 'create':
            const uuid = require('node-uuid')
            event.payload.Item.id = uuid.v1()
            dynamo.putItem(event.payload, () => {
                context.succeed({
                    "id": event.payload.Item.id
                })
            })
            break
        default:
            context.fail(new Error('Unrecognized operation "' + operation + '"'))
    }

}
```

> 其實蠻像我們平常在`redux`中處理對應的 action type 的 `reducer`

這裡建立了一個 `DynamoDB` 的 client，簡單的來說，我們會把 `event.payload` 這個 object，

新增成 Table 裡的一個新 item，並且給它一個唯一的 `id`，

畢竟是 Primary key 嘛！ 

> 如果你不熟悉 Database 的基礎理論，Primary key。

> Primary key 就是我們拿來識別這個 item 在這個表中是唯一的「身分證」，

> 在這裡我們是用 `id`來作為我們的 Primary key。

那這個 `event`又是怎麼來的呢？

首先我們要了解的是 Create 這個動作對應到的 http method 是 `POST`，

所以當我們在對同一個 url 執行 `GET` 跟 `POST`時，

雖然 call 的是同個 function（或者更精確地說，是同一個 Endpoint）。



在 `posts` 資料夾底下，可以看到一個 `s-function.json`，

這個檔案中放著的是關於我們在進入 `handler.js`時相關的 config。

當然也包括了前面說到的 `event`。

先直接看到 `endpoints` 這個 attribute，裡面有許多個物件，

預設的是這個：

```js
{
      "path": "posts",
      "method": "GET",
      "type": "AWS",
      "authorizationType": "none",
      "authorizerFunction": false,
      "apiKeyRequired": false,
      "requestParameters": {},
      "requestTemplates": {
        "application/json": ""
      },
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {
            "application/jsoncharset=UTF-8": "Empty"
          },
          "responseTemplates": {
            "application/jsoncharset=UTF-8": ""
          }
        }
      }
}
```

這裡有好多東西，

假如我們要在裡面定義我們對每個 endpoint 的長相，誰不發瘋呢？

眼尖的你應該看到了有 `template`這個字眼，

而剛剛送進來的 `event` 正是一個 http request，

所以我們要做的事情已經呼之欲出了，就是在`requestTemplates`加上我們指定的 template 名稱，

就能根據這個 template 生出我們想要的 event 。

在 `endpoints` 中加上了這個新的 object：

```js
{
      "path": "posts",
      "method": "POST",
      "type": "AWS",
      "authorizationType": "none",
      "authorizerFunction": false,
      "apiKeyRequired": false,
      "requestParameters": {},
      "requestTemplates": "$${requestCreatePostTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {
            "application/jsoncharset=UTF-8": "Empty"
          },
          "responseTemplates": {
            "application/jsoncharset=UTF-8": ""
          }
        }
      }
}
```

當進入這個 api 時(path 沒有改變)，使用 POST method時，

我們的 request 會照著`requestCreatePostTemplate`這個 template 走

> $${requestCreatePostTemplate} 是特殊的語法，

> 讓 serverless 知道這是個 template 名字，而不是一般的 string。

所以我說，那個 tempalte 呢？

這裡要在 `posts` 底下新增 `s-templates.json`，

所有的關於 lambda function 的 template 都會放在這裡。

接下來我們就可以設計我們的 request（event）的長相了：

```js
{
    "requestCreatePostTemplate": {
        "application/json": {
            "operation": "create",
            "tableName": "posts",
            "payload": {
                "Item": {
                    "content": "$input.json('$')"
                }
            }
        }
    }
}
```

這裡比較讓人疑惑的是 `$input.json('$')`是什麼，

這其實是跟 API Gateway 比較有關係的 template 語法，

而不是 serverless 這個框架底下的。

> This function evaluates a JSONPath expression and returns the results as a JSON string.
> For example, $input.json('$.pets') will return a JSON string representing the pets structure.

簡單的說，他會將 input 轉成一個 json-like string，

更棒的地方是他可以像我們平常 access 底下的 attribut 那樣去找底下的東西：

（就是所謂的 [json path](http://goessner.net/articles/JsonPath/)）

像是 `$.pets` 就是將我們吃到的 input object底下`pets` 對應到的東西，

轉成 string。

> [Amazon API Gateway: Mapping template reference](http://docs.aws.amazon.com/zh_cn/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)

> 想瞭解更多關於 Template 的話可以參考 serverless framework 的文件：

> [Template & Variable](http://docs.serverless.com/docs/application-architectures)

接著回到一開始的 `handler.js`，

就可以把跟 `event` 有關的東西與我們前面 template 裡面所做的 config 連接起來了：

```js
module.exports.handler = (event, context) => {
    console.log('Event: ', display(event))
    console.log('Context: ', display(context))
    const operation = event.operation
    if (event.tableName) {
        event.payload.TableName = event.tableName
    }
    switch (operation) {
        case 'create':
            const uuid = require('node-uuid')
            event.payload.Item.id = uuid.v1()
            console.log('Payload: ', display(event.payload))
            dynamo.putItem(event.payload, () => {
                context.succeed(event.payload.Item)
            })
            break
        default:
            context.fail(new Error('Unrecognized operation "' + operation + '"'))
    }
}
```

這時候可以部署了！

部署完成之後我們需要試試有沒有成功，必須要打開 API Gateway，

一進去就可以看到對應 project 名稱的 api，

點進去能看到我們現在有哪幾個 api 可以用（url）。

可以把 API Gateway 想像成我們平常使用的 `router`，

Gateway 會把要執行的 endpoint 接到對應的 url 上。

點擊 `/posts` 底下 `POST` method 的 integration request ，

在 Body Mapping Templates 可以看到對應的 template：

```js
{"operation":"create","tableName":"posts","payload":{"Item":{"content":$input.json('$')}}}
```

那，要怎麼測試呢？

我習慣用 postman，算是一個測 api 相當好用的工具，

找到`serverless-demo`這 project 底下對應的 `stages` ，

選擇當前對應的 stage（預設應該是 dev），

然後選擇`Export as Swagger + Postman Extensions` 這個選項，

會下載一個 json ，裡面把你所有建立的 request 都包好好的。

接著就能在 postman 中 import ，就能直接使用了。

首先當然是先測試原先的 `GET` method，理論上來說應該要丟出 error，

因為送進來的 request(event)，它的 `operation`是 `undefined`：

```js
{
  "errorMessage": "Unrecognized operation \"undefined\"",
  "errorType": "Error",
  "stackTrace": [
    "module.exports.handler (/const/task/handler.js:28:26)"
  ]
}
```

非常的好。

接著是`POST`：


```js
{
  "errorMessage": "Process exited before completing request"
}
```

居然噴錯了，所以我們要再度到 CloudWatch 去看一下 log，

看起來 `event` 的樣子是對的，但往下一看就找到了這個錯誤：

```
Cannot find module 'node-uuid'
```

我們在根目錄雖然有`package.json`，

但是目前對於底下的 `handler.js` 而言，

它對根目錄是完全一無所知的，那該怎麼做呢？

在`s-function.json` 中的 `handler` 改成 `functions/posts/handler.handler`，

我們能在這裡決定 function 要對整個 project 的權限到哪裡，

像這裡就會一直延伸到根目錄，所以我們在根目錄所安裝的 package，

自然到了`posts`底下也吃得到了。

假如仍然沒有辦法動到 dynamodb 的話，

就要到 `s-resources-cf.json` 更改設定

在`IamPolicyLambda.Properties.PolicyDocument.Statement`底下加上：

```js
{ 
    "Effect": "Allow",
    "Action": ["*"],
    "Resource": "arn:aws:dynamodb:${region}:*:table/*"
}
```

再去 Postman 執行一次，

DynamoDB 的 Table 裡面就會出現新一筆的資料了（一個新的 Item）。


## Read an item

- 我們剛剛已經可以在 DynamoDB 裡面新增資料，自然要有辦法拿出來才是。

第一步一樣是從 `handler.js` 裡面直接去做更改：

> 為什麼每次都從 `handler.js`開始是因為這邊是最符合邏輯的地方，

> 其他都比較特定的 config 問題

```js
switch (operation) {
    case 'create':
        const uuid = require('node-uuid')
        event.payload.Item.id = uuid.v1()
        console.log('Payload: ', display(event.payload))
        dynamo.putItem(event.payload, () => {
            context.succeed(event.payload.Item)
        })
        break
    case 'read':
        dynamo.getItem(event.payload, context.done)
        break
    default:
        context.fail(new Error('Unrecognized operation "' + operation + '"'))
}

```

接著要到 `s-function.json` 裡面去加上對於 parameter 的設定，

以及加上 template：

> 在 GET method 的底下

```js
"requestParameters": {
        "integration.request.querystring.id": "method.request.querystring.id"
      },
"requestTemplates": "$${requestReadPostTemplate}"
```

最後則是 template：

```js
"requestReadPostTemplate": {
    "application/json": {
        "operation": "read",
        "tableName": "posts",
        "payload": {
            "Key": {
                "id": "$input.params('id')"
            }
        }
    }

}
```

> 假如你好奇為什麼要用`Key` 的話，

> 可以參考 DynamoDB js sdk 的 [github](https://github.com/awslabs/dynamodb-document-js-sdk)

> 與 mongodb 的 query 非常相似

因為我們在 handler 中用了 `context.done`，

這裡其實是個 callback function，等到 `getItem` 結束後，

才會執行 `context.done` ，

並且會依序傳入 `error`、`data`兩個 object，

所以回傳的 response 會是像這樣的一整個 item：

```js
{
  "Item": {
    "id": "3caaeb80-1ebf-11e6-81a9-21cf9c171332",
    "content": {
      "message": "Hello world again!"
    }
  }
}
```

有時候我們並不想讓使用者知道這麼多，

所以可以使用 response template，

這裡就能看到前面說的 json path 的用處：

```js
// s-function.json
"responseTemplates": "$${responseReadPostTemplate}"
```

```js
// s-templates.json
"responseReadPostTemplate": {
    "application/json": {
        "post": {
            "id": "$input.path('$').Item.id",
            "content": {
                "message": "$input.path('$').Item.content.message"
            }
        }
    }
}
```

## Update an item

Update 跟 Read 的做法其實已經大同小異，

一樣是把查詢用的 Key 放在 `params` 中，

這裡我們一樣把整包 payload 都丟進來。

```js
dynamo.putItem(event.payload, (err, data)=> {
    context.succeed(event.payload)
})
```

看起來只是改成使用 `putItem` 而已，

但其實這邊的 template 有點小小的改變。

```js
"requestUpdatePostTemplate": {
    "application/json": {
        "operation": "update",
        "tableName": "posts",
        "payload": {
            "Item": {
                "id": "$input.params('id')",
                "content": "$input.json('$')"
            }
        }
    }

}
```

這樣子的好處就是在更新時，只要在 params 輸入指定的 `id`，

其餘要更新的部分就是放在 `body`裡面。

> 這裡的 `PUT` 並不是 partial 的更新，

> 而是整個會替換掉，符合它原本 HTTP method 對應的行為

至於`s-function.json` 裡面要怎麼改，這有點太 trivial ，

就不放上來了。

## Delete an item

刪除一個 item，要做的事情比 update 單純多了，

基本上只要指定好 Key，一切就已經結束了：

```js
dynamo.deleteItem(event.payload, context.done)
```

```js
"requestDestroyPostTemplate": {
    "application/json": {
        "operation": "destroy",
        "tableName": "posts",
        "payload": {
            "Key": {
                "id": "$input.params('id')"
            }
        }
    }
}
```

## List items

除了以上的 CRUD 之外，

列出一定數量的 items 也是一個相當常見的需求。

```js
dynamo.scan(event.payload, context.done)
```

```js
"requestListPostTemplate": {
    "application/json": {
        "operation": "list",
        "tableName": "posts",
        "payload": {}
    }
}
```

最後的 Response template 會用到 `foreach` 語法，

坦白說這裡我壓根不想去理解這裡的意義是什麼，

我寧願在需要的時候再去查文件就好，

因為我相信這種夭壽的語法遲早會被改掉的：

```
"responseListPostTemplate": "{\"posts\" : [#foreach($post in $input.path('$').Items){\"id\" : \"$post.id\",\"content\" : { \"message\":\"$post.content.message\" }}#if($foreach.hasNext),#end #end ] }"
```

# Conclusion

現在大概知道，

為什麼當初開始學的時候網路上沒什麼好的教學文了，

因為 config 的設置真的是挺複雜的，

不過我想這一篇這樣記錄下來，應該能讓許多人省下走冤枉路的時間。

對於一個程式開發者來說，學習東西的時間就是最大的成本，

我想 serverless 不管對於前後端來說，

都是一項很超值的投資。

因為大部分時候，我們都不需要開一整台機器來完成你想做的事情。

在完成這篇之後，可以做什麼練習呢？

你可以試著把你原本在 EC2 上 host 的服務，

轉移成 serverless 架構。

> 光想就覺得超難的

或者是把一些 routine 的工作，用 serverless 的方式去做，

當你越過前面那些雞巴毛 config 後，

你會發現開發和部署上帶來的效率令你吃驚。

# References

- [Serverless node dynamodb example](https://github.com/markusklems/serverless-node-dynamodb-example)

- [serverless framework document](http://docs.serverless.com/docs/)

- [解析 DynamoDB](http://history.programmer.com.cn/11081/)

- [Amazon API Gateway: Mapping template reference](http://docs.aws.amazon.com/zh_cn/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)

- [Micro service and Nano service](http://alexfalkowski.blogspot.tw/2013/12/micro-and-nano-services.html)