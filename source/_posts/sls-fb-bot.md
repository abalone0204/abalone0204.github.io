---
title: 實作 Serverless 的 facebook messenger bot
tags:
  - fb
  - bot
  - serverless
date: 2016-05-29 11:55:40
---


上禮拜簡單介紹了一下 serverless 的起手式後，

我想再舉個更接近實際應用的例子，

儘管並不是所有的應用都適合 serverless 的架構，

但聊天機器人(chat bot)是一個相當好的例子，

且讓我稍後再說明為什麼。

今天就結合一下很實用的粉絲頁回覆機器人以及 serverless 。

> 你可以把聊天機器人想成是你粉絲頁自動回覆的員工

> 或是進行一些簡單的操作

> 而聊天機器人流行起來的原因正是因為 mobile 裝置上的介面，

> 並不能滿足於現代人操作的所有需求，

> 聊天的介面解放了我們在小框框裡做事的限制。

> 或者是你是小小公司的開發者，需要一個助理來幫你做很無腦或繁瑣的事情，

> 再講下去可能要一篇了，如果你對這個主題有興趣，

> 可以看看 [灣區日報是如何運作的](https://wanqu.co/blog/2015-05-24-behind-the-scenes.html)

<!-- more --> 

# Introduction

![bot](http://i.imgur.com/JmVfjQ5.png)

微軟、line、slack 都出了，

臉書當然也要 bot 來幫我們處理一些事情。

> 相當不建議直接照著貼，可以先看看我的前一篇文章，

> 至少現在敢大膽的說是目前最詳盡的 serverless 繁體中文入門教學：

> [淺析 serverless 架構與實作](http://abalone0204.github.io/2016/05/22/serverless-simple-crud/)

這一篇筆記裡面會介紹如何把一個 facebook 粉絲專頁的 bot，

用 serverless 的方式架起來。

這個 bot 能夠：

- 處理粉絲專頁接收到訊息的 events

- 執行對應的動作或回傳訊息

為了保持簡單，並且專注在 messenger bot 本身，

我不會用到其他服務的 events，像是 DynamoDB 或是 S3 之類的，

但其實只要能掌握收訊息，以及對應訊息做出動作，

基本上就掌握了搭配其他功能的 interface 了 :D

假如你是個懶得看文章的人，我一樣把 code 放在 github 上面了：

- [serverless-facebook-messenger-bot-starter](https://github.com/abalone0204/serverless-facebook-messenger-bot-starter)

有幫助到你的話給星星打賞，有問題的話也歡迎提 issue 或直接告訴我。


# Catalogue

- [Why serverless](#Why-serverless)

- [Implementation](#Implementation)

    - [申請作業](#申請作業)

    - [Serverless 環境建構](#Serverless-環境建構)

    - [https](#https)

    - [Deployment 部署](#Deployment-部署)

    - [Subscribe to fan page](#Subscribe-to-fan-page)

    - [Reply - 回覆訊息](#Reply-回覆訊息)

    - [更複雜的回覆訊息格式](#更複雜的回覆訊息格式)

- [Conclusion](#Conclusion)

# Why serverless

為什麼我認為 chat bot 是一個非常適合 serverless 架構的運用？

想想我們平常聊天，訊息也都不會馬上回嘛！

所以我們其實不需要那麼真正的「real time」，

而且只有在有人丟訊息時，lambda 才會幫我們運算，

省下了不少機器閒置在那的費用。

- 對於延遲時間的容忍度高：
    
    - 容忍了 serverless 的缺點

- 有使用才收費：
    
    - 省錢、加上使用情境相當符合

- 簡單的運算

    - AWS lambda 運算時間不能超過五分鐘，否則會被強制結束，但這種簡單的文字回覆，通常處理不會超過五分鐘...吧

    - 當然如果你要跑什麼類神經網路，那我會建議那些運算邏輯可以放在真正的 server 上

> 題外話是這篇 [面試遇到 用 deep learning 解 fizzbuzz](http://joelgrus.com/2016/05/23/fizz-buzz-in-tensorflow/)

> 看到後面超好笑 XD

- `https`: facebook 的 bot 會需要有 https ，通常可以透過 CloudFlare 免費申請一個，但假如你使用 lambda 的話，原生給你的連結就是 https 的。
    

# Implementation

同樣的，因為我認為介面隨時會改變，

所以我不做截圖的 step by step 。

## 申請作業

- 申請 facebook app、一個要用到的粉絲頁

> 申請的類型有 ios、android 什麼的，

> 先選網頁，然後網址可以亂打一通，這對之後沒有影響

- 到 facebook 的 app 控制台

- 在控制列選擇新增產品

- 選 Messenger Expression

- 會看到一個新的 Messenger 跑出來了，選它

- 接著可以選擇你要把你的 bot 安置的粉絲頁，選擇後會得到一個權杖。

> 我覺得權杖是一個一聽會覺得「啥？」的命名，

> 不過它的意思就是你能夠讓 bot 藉由這個「權杖」，

> 取得在你粉絲頁發文或是發訊息的「權利」

- 接下來選擇 setup webhooks

> 什麼是 Webhook？

> 你可以把它看成是一種 back-end 到 back-end 之間的通知，

> 最常見的例子就是 CI 了

> e.q：今天在 github 上送了一個 commit，

> webhook 就會把這邊更新的訊息帶去給 CI server，

> CI server 收到後就會開始跑後續的流程

> hook，就是鉤子，在網路上把訊息以及收到訊息要執行的行為鉤住，

> 帶到別的地方(callback url)去的就是 webhook

- 到這裡我們就可以去設置一下環境了

## Serverless 環境建構

為了驗證我們的 callback 是不是正確的，

facebook 這邊會去做驗證，

確認它送來的`hub.verify_token`跟你粉絲專頁的權杖一樣時，

就會把 request 中的`hub.challenge`送回來。

> 這裡有個小雷是我們要送回來的值是 integer，不是 string

官方的例子大概長這樣：

```js
// Node.js Example
app.get('/webhook', function (req, res) {
  if (req.query['hub.verify_token'] === <YOUR_VERIFY_TOKEN>) {
    res.send(req.query['hub.challenge']);
  } else {
    res.send('Error, wrong validation token');    
  }
})
```

接著就一如往常的開一個 serverless 專案，

建立一個 handler function。

> 參考這邊[serverless setup 開發環境的建置](http://abalone0204.github.io/2016/05/22/serverless-simple-crud/#Setup-開發環境的建置)

```
serverless function create functions/bot
```

然後在來看程式的進入點：

```js
module.exports.handler = function(event, context) {
    
    const operation = event.operation
    

    switch (operation) {
        case 'verify':
            const secret = event.secret
            const verifyToken = event["verify_token"]
            if (secret === verifyToken) {
                context.succeed(parseInt(event["challenge"]))
            } else {
                context.fail(new Error('Unmatch'))
            }
            break
        default:
            context.fail(new Error('Unrecognized operation "' + operation + '"'))
    }

}
```

> `operation`這個屬性是為了後續的動作，

> 不管對這個 callback url 呼叫東西，

> 都會進入這個 `handler.js` 但是我們必須有不同的動作，

> 我認為這裡都是屬於在 bot 執行動作的邏輯之下，

> 所以將它們放在同一個 handler.js 中，你完全可以有不同的編排方式 :)

`event` 裡的東西哪裡來呢？

`event`其實就是 request，serverless 是個 event-driven 的架構，

我們可以在 `s-templates` 裡面去設置 template，

這裡有個 tricky 的問題，就是要怎麼處理權杖？

有兩種方法，

一種是在本地端用 module export 的方式解決，

另一種則是用 aws 的 env variable。

首先先看用 aws 的 variable 怎麼解決

```
sls variables set -k KEY -v VALUE -s STAGE -r REGION
```

然後我們就可以在 template 中使用 ${KEY} 的語法來拿到 variable，

這裡要注意你是不是在每個不一樣的 stage 以及 region 都設置了 variable。

要檢查的話可以進去自動生成的 `_meta` 資料夾看。

> `_meta` 是自動被 git 給忽略的

接下來到 `s-function.json` 裡面設定 request 的 template，

把 callback

```json
"requestTemplates": "$${apiGetCallbackTemplate}",
```

再來看 `template` 長什麼樣子：

```json
{
    "apiGetCallbackTemplate": {
        "application/json": {
            "operation": "verify",
            "secret": "${fb_secret_key}",
            "verify_token": "$input.params('hub.verify_token')",
            "challenge":"$input.params('hub.challenge')"
        }
    }
}
```

假如你還不太熟悉 serverless，

這裡就是在描述剛剛 `handler` 中`event`的長相：

```js
{
    "secret": "FB_SECRET_KEY",
    "verify_token": "VERIFY_TOKEN",
    "challenge":"CHALLENGE_CODE"    
}
```

你可能會覺得對方如果知道你的 callback url 那不就顯示出你的 secret 了嗎？

其一是這個連結不會對外，而你也可以限制 request 的來源，

而這也是為什麼要加上這一段的原因：

```js
if (secret === verifyToken) {
    context.succeed(parseInt(event["challenge"]))
} else {
    context.fail(new Error('Unmatch'))
}
```

如果沒有 secret 跟 verifyToken 沒有相等的話，

會直接結束，並且返回 error。


假如你不熟悉 aws 也不想接受這樣的做法的話，

你可以在本地新建一個`secret.js`

```js
export const secret = "FB_SECRET_KEY"
```

然後把這支檔案 `.gitignore` 就行了，

不過這其實算是一種 hack 的方式，並不是一個很漂亮的做法。

## https

假如你要自己 host 一個服務來放 bot 的話，

還要去額外申請 https，但如果你用 serverless，

搭配 api gateway 就直接幫你避免掉了這個問題


## Deployment 部署

```
serverless dash deploy 
```

`function - callback` 跟 `endpoint - callback` 都選起來，

部署上去之後會返回一個網址，

當我們對這個網址送一個帶有 http method 為 GET 的 Requst 時，

就會進入我們剛剛看到的 `handler.js` 中執行東西。

- setup wehook

最後就是把返回的那個網址貼在 callback url 那裡，

再把權杖給貼上去：

![webhook setup](https://scontent-tpe1-1.xx.fbcdn.net/t39.2178-6/12057143_211110782612505_894181129_n.png)

(下面的欄位我都會全勾起來 XD)

正確的方式應該是在 back-end 上放上 secret（這裡指權杖），

facebook 會送個 request 到你的 callbakc url 去，

並且看看在 params 中的 `hub.verify_token` 是不是等於你放上去的 secret，

如果是的話，再把 params 中的 `hub.challenge` 當作 response 丟回來，

facebook 就會判定你這個 webhook 通過認證，

後續才能繼續進行下去。

## Subscribe to fan page

有兩種方法可以去「監聽」粉絲專頁收到訊息的 event。

- 以在 facebook app 操作的後台上選擇你要訂閱哪個粉絲專頁收到的訊息

假如你寫過 rx，會知道 subscribe 可以監聽 event 是否進來，

接著我們會去做對應的動作。

假如你沒寫過 rx，~~那你應該去學一下~~。

簡單說就是當我們監聽的粉絲專頁收到訊息時，

剛剛設定的 webhook 會送一個 post method 的 request，

而我們可以做出對應的行為，這裡通常就是返回一些訊息，

facebook 的 messenger 還可以回傳附件之類的。

官方給的 demo code 長這個樣子，先只要大略掃過一遍就好，

後面會更詳細解說這裡在幹什麼，

畢竟第一次看到的時候我也不知道這到底在幹嘛：

```js
app.post('/webhook/', function (req, res) {
  messaging_events = req.body.entry[0].messaging; // 拿到 request 中的訊息
  for (i = 0; i < messaging_events.length; i++) {
    event = req.body.entry[0].messaging[i];
    sender = event.sender.id; // 送訊息人的 id
    if (event.message && event.message.text) {
      text = event.message.text;
      // Handle a text message from this sender
    }
  }
  res.sendStatus(200);
});
```

唯一知道的是我們送訊息時，會丟一個 POST reqeust 給 webhook，

雖然最後得到了一個 `sender`（訊息的發送者），以及傳送的`text`訊息，

還是有點搞不懂到底在做什麼，像遇到這種情形時，

把東西 log 出來就對了。

所以第一個目標就是來觀察一下 facebook 到底會送一些什麼東西過來。

先把 post method 的 template 建出來

```js
{
    "apiPostCallbackTemplate": {
        "application/json": {
            "operation": "reply",
            "body": "$input.json('$')"
        }        
    }
}
```

`handler.js` 中其實 succeed 傳回的結果是什麼都沒差，

重要的是我們能看到傳過來的 request，要把它 log 出來

這是我們在寫 code 時常做的 debug 方法，

就算 serverless 其實也沒有不同 XD

```js
// inside the handler function
function display(object) {
    return JSON.stringify(object, null, 2)
}

console.log('Event: ', display(event))
switch(operation) {
    case 'reply':
        context.succeed(event)
        break
}
```

接著我們到 facebook 上丟給我們剛剛創的粉絲專頁一些訊息，

假設我們密他然後說個：「Hello bot 」

到 AWS Cloud Watch 上面就可以看到返回的 body 長這個樣子，

可以快速的掃過一次（大寫的是是代表一些 id，你懂的）：

```js
{
    "object": "page",
    "entry": [
        {
            "id": ENTRY_ID,
            "time": 1464447058752,
            "messaging": [
                {
                    "sender": {
                        "id": SENDER_ID
                    },
                    "recipient": {
                        "id": RECIPIENT_ID
                    },
                    "timestamp": 1464447058667,
                    "message": {
                        "mid": "mid.1464447058507:7548866c81ec168b21",
                        "seq": 4,
                        "text": "Hello bot"
                    }
                }
            ]
        }
    ]
}
```

看到這個之後，比較能知道 facebook 的 sample code 在幹嘛，

而不是單純的 copy and paste。

再上一次 sample code 來對照一下

```js
app.post('/webhook/', function (req, res) {
  messaging_events = req.body.entry[0].messaging;
  for (i = 0; i < messaging_events.length; i++) {
    event = req.body.entry[0].messaging[i];
    sender = event.sender.id;
    if (event.message && event.message.text) {
      text = event.message.text;
      // Handle a text message from this sender
    }
  }
  res.sendStatus(200);
});
```

看起來 facebook 的工程師為了保留開發上的彈性，

所以加上了一些目前看起來有點冗的東西，

我們可以選擇一開始就把 `messaging_events` 在 template 裡面拿出來，

或者是一樣拿回整個 body，不過為了說明方便，

還是照它原本的格式走。

總之，理解後就能開始試著把它改成 serverless 的模式了：

（真的是幾乎長得一模一樣）

```js
switch (operation) {
    case 'reply':
    const messagingEvents = event.body.entry[0].messaging
    messagingEvents.forEach((messagingEvent) => {
        const sender = messagingEvent.sender.id
        if (messagingEvent.message && messagingEvent.message.text) {
            const text = messagingEvent.message.text
            // Handle a text message from this sender
        }
    })
    break
}
```

為什麼要拿 `sender` 以及 `text`呢？

原因就是待會回覆訊息會需要用到。

## Reply - 回覆訊息

回覆訊息要用到我們之前的能登入粉絲頁的「密碼權杖」，

假如你是用 variable 解決的話，這部分會簡單很多。

只要把剛剛在 callback url 的 `fb_secret_key` copy 過去就好了：

```js
"apiPostCallbackTemplate": {
    "application/json": {
        "secret": "${fb_secret_key}",
        "operation": "reply",
        "body": "$input.json('$')"
    }        
}
```

一樣先來看一下 sample code 是怎麼做的：

```js
var token = "<page_access_token>";

function sendTextMessage(sender, text) {
  messageData = {
    text:text
  }
  request({
    url: 'https://graph.facebook.com/v2.6/me/messages',
    qs: {access_token:token},
    method: 'POST',
    json: {
      recipient: {id:sender},
      message: messageData,
    }
  }, function(error, response, body) {
    if (error) {
      console.log('Error sending message: ', error);
    } else if (response.body.error) {
      console.log('Error: ', response.body.error);
    }
  });
}
// 實際傳送訊息
sendTextMessage(sender, "Text received, echo: "+ text.substring(0, 200));
```

沒錯，這裡根本就可以直接拿來用了，

我們先求有再求好：

```js
module.exports.handler = function(event, context) {
    const operation = event.operation
    const secret = event.secret

    function sendTextMessage(sender, text) {
        const messageData = {text: text}
        request({
            url: 'https://graph.facebook.com/v2.6/me/messages',
            qs: {
                access_token: secret
            },
            method: 'POST',
            json: {
                recipient: {
                    id: sender
                },
                message: messageData,
            }
        }, (error, response, body) => {
            console.log('GET response', response);
            context.succeed(response);
            if (error) {
                context.fail('Error sending message: ', error);
            } else if (response.body.error) {
                context.fail('Error: ', response.body.error);
            }
        })
    }
    
    switch (operation) {
        case 'reply':
            const messagingEvents = event.body.entry[0].messaging;
            messagingEvents.forEach((messagingEvent) => {
                const sender = messagingEvent.sender.id
                if (messagingEvent.message && messagingEvent.message.text) {
                    const text = messagingEvent.message.text;
                    sendTextMessage(sender, "Text received, echo: "+ text.substring(0, 200))
                }
            })
            break
        default:
            context.fail(new Error('Unrecognized operation "' + operation + '"'))
    }

}
```

在執行 `sendTextMessage` 時，

裡面的 `request` 會是非同步的，

也就是說在後續的流程裡如果你讓整個 function 提早結束的話，

訊息將不會被傳送。

不過 user 一進來，其實不會知道 bot 有哪些功能，

我們可以設定對話剛開始的開場白，只要在執行這行：

```
curl -X POST -H "Content-Type: application/json" -d '{
  "setting_type":"call_to_actions",
  "thread_state":"new_thread",
  "call_to_actions":[
    {
      "message":{
        "text":"Hi, 歡迎來到 Serverless Maniac。我是機器人，輸入 help 來看有什麼指令可以用吧"
      }
    }
  ]
}' "https://graph.facebook.com/v2.6/<PAGE_ID>/thread_settings?access_token=<FB_SECRET_KEY>"
```

> `FB_SECRET_KEY`就是前面提到的密碼權杖，`PAGE_ID` 是你粉絲頁對應的 id，

出來結果大概就是這樣子

![demo](http://i.imgur.com/hruiXeO.jpg)


## 更複雜的回覆訊息格式

- facebook 也提供一些更 fancy 的訊息格式 

- 針對特定的訊息去做動作

比起一般的小編回覆訊息，這裡能夠藉由 messenger platform 提供的 API，

回覆一個更像 app 的訊息模板、提供更棒的 UX，

啊！這樣講好抽象，直接看一下成果的話大概是這樣子：

![struc demo](https://media.giphy.com/media/xT4uQAs24rrYQDjNuw/giphy.gif)

> 沒錯，就是做了一個自己 blog 的 feeds

剛剛在 `sendTextMessage` 裡面會把 `text`再額外包一層處理，

可見這裡是保留了其他彈性，

往後翻一下文件就會看到我們可以自訂訊息的模板。

```js
const text = messagingEvent.message.text;
const messageData = genMessageData(text)
sendTextMessage(sender, messageData)
```

在 `genMessageData` 裡面：

> 不要被長度嚇到了，你可以對照圖片中的字，

> 跟下面程式碼做對照，其實都只是在處理 elements 裡面一個個 object 而已

```js
if (text === 'feeds') {
    return {
        "attachment": {
            "type": "template",
            "payload": {
                "template_type": "generic",
                "elements": [{
                    "title": "淺析 serverless 架構與實作",
                    "subtitle": "May 22, 2016",
                    "image_url": "http://i.imgur.com/lP3wcnh.jpg",
                    "buttons": [{
                        "type": "web_url",
                        "url": "http://abalone0204.github.io/2016/05/22/serverless-simple-crud/",
                        "title": "open"
                    }]
                }, {
                    "title": "Saga Pattern 在前端的應用",
                    "subtitle": "May 14, 2016",
                    "image_url": "https://upload.wikimedia.org/wikipedia/zh/3/37/Adventure_Time_-_Title_card.png",
                    "buttons": [{
                        "type": "web_url",
                        "url": "http://abalone0204.github.io/2016/05/14/redux-saga/",
                        "title": "open"
                    }]
                },
                {
                    "title": "淺入淺出 Generator Function",
                    "subtitle": "May 8, 2016",
                    "image_url": "http://www.rumproast.com/images/uploads/shallow_end_thumb.jpg",
                    "buttons": [{
                        "type": "web_url",
                        "url": "http://abalone0204.github.io/2016/05/08/es6-generator-func/",
                        "title": "open"
                    }]
                },
                {
                    "title": "Super tiny compiler",
                    "subtitle": "Apr 25, 2016",
                    "image_url": "https://cloud.githubusercontent.com/assets/952783/14413766/134c4068-ff39-11e5-996e-9452973299c2.png",
                    "buttons": [{
                        "type": "web_url",
                        "url": "http://abalone0204.github.io/2016/04/25/Super-tiny-compiler/",
                        "title": "open"
                    }]
                }
                ]
            }
        }
    }
}
```


# Conclusion

截至目前為止，我們已經理解了怎麼接收和傳送訊息，

對我來說這是一個比 slack 更輕量的小助理，

其實搭配 DynamoDB 或是其他 backend 就可以做到 schedule 的效果。

同時我認為 bot 並不是拿來取代小編的，

可以將一些常問的問題和解答建在 bot 裡面，

讓小編不用再去回一些重複的問題，專注在寫出更好的文案，

以及更急迫需要回應的客戶上面。

> 可以選擇搭配 [hubot](https://hubot.github.com/) 來處理各種訊息，

> 以及對應的動作。

> 不過仍然要強調一下，這篇筆記著重在如何建立一個這樣的 interface：`收訊息 => 執行動作`

> 另外，把程式邏輯全部都放在 `handelr.js`，只是為了說明方便，

> 你可以選擇自己喜歡的方式來建構 bot。

最後，額外提醒一下 XD

目前完成的 bot 只能夠跟你個人通話而已，

假如你想讓其他人也看到的話，

必須到 facebook app 的控制台通過 facebook 的審核後才行，

希望大家能做出許多好玩的粉絲專頁應用 XD


# References

- [Facebook Messenger Platform](https://developers.facebook.com/docs/messenger-platform)

- [淺析 serverless 架構與實作](http://abalone0204.github.io/2016/05/22/serverless-simple-crud/)

