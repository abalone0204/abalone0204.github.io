---
title: 我的前端工程師之路
date: 2016/1/18
tags: [career, front-end]
---

「這個畫面幫我稍微調一下應該沒有很難吧？」

<!--more-->

Hi 大家，先簡介一下自己背景：

- 非資工資科資管系(也就是所謂的非本科系)

> 其實我覺得第一點不是很重要、我也很討厭強調，

> 但偏偏就是很常被問，

> 要不然就是別人聽到你大學是XX系就會：「蛤？」

> 索性就把它列出來了。

- 興趣使然的前端工程師，擅長一鍵跑版

- 沒上過資策會、巨匠，或任何各種職訓班，但買過 tree house 的課程和幾本書

- 學習時間：一年半（從碼盲到現在終於可以改一些 code）

- 稍微熟一點的技能： JavaScript, CSS, html, React.js

- 預計未來要學的東西：Haskell, golang, Angular(2.0), Rx.js

- 這是之前相關的[專訪](http://westudent-blog.sudo.com.tw/denny-sudo.html)，但我一直都想自己寫一篇，也不是 William 寫得不好，但總覺得哪裡不對勁，也許自幹就是一種工程師的浪漫吧！

> 後來想想，也許是整篇文章太強調「本科」影響不大

> 事實上我想說的是：「非本科不該是阻擋你寫程式的理由。」


- 總之這篇就是來介紹一下我是怎麼慢慢上手這個職業，中間會提到一些我覺得很棒的學習資源，和吸收新知的方式。

- 說到吸收新知，目前首推[碼天狗](http://weekly.codetengu.com/)，它讓我禮拜一早上都會很焦慮的重新整理，大家可以感受一下。


# Porjects

- 這是我簡介自己做過 projects 的[slide](http://slides.com/dennyku/deck-1#/)

- 會放這個是因為我本來要買月費專案($7/month)，卻手滑買到年費專案($70/year)，gan，只好多多利用它了。

---

# Get started

其實我本來立志成為一個 Data Scientist ，

只是不小心被擺到前端的位置上去......

回顧這一年半的旅程，前端的東西真的太多太雜了，

更容易完全只知其然而不知其所以然的就開始用某個新框架、library，

所以對我來說，**「學什麼」是副課題，「不學什麼」才是真正的關鍵**。

---

因為我前端工程師的路還沒走完，

所以應該在我退休或換職業（去賣雞排）之前，

都會繼續寫下去。

目前寫完三點：

1. 非本科系 v.s 本科系

2. 從哪裡開始學習？

3. 前端工程師該懂後端嗎？

---

## 1. 非本科系 v.s 本科系

就來說說**「本科系」**來到底有沒有差。

首先，我們都知道學校裡的課程，

很少是真的專注在所謂「**前端工程**」上；

這是可以理解的，一來因為前端變化太快，

學期初才在說好棒棒的東西，

到了學期末可能就變 deprecated了。

所以這就代表非本科系跟本科系的人站在相同的學習立足點上了嗎？

No，你得面對現實，本科生就是有他的優勢在。

這裏要講個實習的故事。

我第一間去實習的新創公司，應徵的是行銷，

CEO 是個自己學習 JavaScript 並且把產品做出來的人，

更重要的是他是個很願意教的人，

在我表示我想朝這方向前進的意願時，

他很大方的說：「如果你對 JavaScript 有興趣可以教你。」

當時還有另一位是資管系的同學也一起，

第一次的作業是用 Angular 做表單的驗證，

怎麼讓使用者不能繼續輸入資料呢？

（當時的我連 JavaScript、html 都不會寫）

我的做法是非常土炮的將 input 換成 div 然後加上紅色的邊框，

另一位實習生則是使用了 disabled 這個 property，就搞定了。

講起來也沒什麼了不起的技巧，但不知道就是不知道。

我問他怎麼會知道有 disabled 這個特性，

他的回答也很簡單：「查文件啊！」

也是這次教訓，我知道要先**查文件**。

講起來蠻白癡，

不過會上 [stackoverflow](http://stackoverflow.com/) 和 google 找答案和看官方文件，

都是最基本的能力。

為什麼他會知道？

很簡單，因為平常他們在寫作業或作專題就需要這個能力。

![rtfm](http://i.imgur.com/B8wms6V.png)

> 既然我們遇到不會的字會查字典，
> 那為什麼我們寫軟體遇到問題時，不需要讀 doc 呢？

而對於整個電腦的理解，非本科系的人絕對也是被甩在幾條街之外，

因為我們不需要修資料結構、演算法，

更別說對於資料庫，

作業系統、計算機結構、計算機組織、編譯器理解的淺薄，

一定要掌握上述這些知識才能寫前端嗎？

這是一個很大的疑問；

但一個了解底下發生什麼事情的人，才會更知道極限在哪裏，

這個絕對是肯定的。

有時候你寫程式時會卡在一個小小的點，想出來之後覺得沒什麼，

而本科系的人能從以前上述課程中的經驗去延伸，

（不管是演算法或是系統相關的事情）

比你更快速得到答案。

畢竟，人家花了那麼多時間了解電腦，

你如果不是天縱英才，要比他們理解電腦就得更努力跟上才行。

這裏推薦一個很棒的課程，[nand2tetris](http://www.nand2tetris.org/)，

上面有很詳細的指示，如果你需要影片和評分系統的話，

coursera 上也有開課了:[https://www.coursera.org/course/nand2tetris1](https://www.coursera.org/course/nand2tetris1)。

這門課會從最基本的 nand(not and) 邏輯閘開始講起，

用模擬器組合出自己的 CPU、記憶體，定義自己的組合語言，

用習慣的程式語言寫出組譯器，

再寫出一個超簡易版的 JVM，最後用一個簡化過後的 Java 語言（真的超簡化），

寫出一個俄羅斯方塊來。

整台電腦、軟體，都是由你一手寫出來的，不覺得很熱血嗎？

而且你終於也能夠看懂這張圖的笑點在哪了：

![gif](https://media.giphy.com/media/3oEdv6pGyOH00ZiRH2/giphy.gif)

當然，如果你在學習途中發現你對系統的東西很有興趣，

那也恭喜你發現新天地啦！

想當初為了所謂堅實的基礎，還跑去圖書館借白算盤來啃，

那又是另一個故事了。


總結一下這一段，

**前端工程師也是軟體工程師**，

對電腦一無所知的人寫出來的軟體，你敢用嗎？

我認為至少玩過一輪 nand2tetris 對於非本科系的人會相當有幫助，

本科系的人來寫前端確實是有一點優勢在，

但這不是認輸的藉口，

而是你必須比別人更努力找方法變強的原因。

另外，

千萬不要以為念研究所的人是只會讀書的書呆子，

比你聰明、比你努力，又比你勇敢的人永遠都多的是。

----

## 2. 從哪裡開始學習？

先來說說「單純」的前端從哪裡開始，

主要分成兩塊：

第一塊是 html 和 CSS：

我以前學習 html 和 CSS 的方法就是把 [w3schools](http://www.w3schools.com/)看完，

不能說有什麼不好，不過真的是看完大部分都忘記，

畢竟很多東西都馬是要用到的時候再去查。

但現在我會推薦 [codecademy](https://www.codecademy.com/)，

邊寫點東西邊學絕對是很有效的學習方式。

而學會基礎後，

要怎麼設計出好維護又乾淨的 html and CSS 那又是另一個很長的故事。


第二塊則是 JavaScript：

坦白說一年半過去，我仍然認為自己在 JavaScript 的知識上很貧瘠。

這裏有篇 [10 個面試時應該要知道的問題](https://medium.com/javascript-scene/10-interview-questions-every-javascript-developer-should-know-6fa6bdf5ad95#.k5wxhl2s1)，

可以探一下自己到底對 JavaScript 理解多少。

這裏如果把教學全部列出來，真的是完全列不完，

但學習的流程是這樣子：

- 掌握了基礎的語法和原則

- 實作練習

- 回頭研究基礎再繼續實作

- 重複以上循環不斷的把你的武器磨的更亮


至於掌握基礎的語法，你可以到以下任一網站，

挑一個你喜歡的，上完基礎 JavaScript 課程：

- [tree house](https://teamtreehouse.com/) 

- [codeschool](https://www.codeschool.com/)

- [egghead.io](https://egghead.io/)

練習一段時間後，你會發現又有好多新工具冒出來了，

這時候你可以先辦個 github 帳號，

首先 watch awesome 這個 repo: [https://github.com/sindresorhus/awesome ](https://github.com/sindresorhus/awesome)，

看一下你喜歡的領域有沒有什麼好東西，

再挑幾個你最有興趣的 repo 按下 watch，

最後再開始訂閱各大框架或社群的 weekly，

接著就是準備被源源不絕的資訊轟炸、不斷的學習和升級。

而值得一提的是， JavaScript 有很多工具可以用，

不管是 library 還是 framework，

學習之前，真的必須想一想：

**「你真的需要用它嗎？」**

舉例來說：

React 的確相當的好用，

但是你的畫面真的有那麼多 state 要處理嗎？

有些人簡單的認為 SPA(Single Page Application)就要用 React，

我得說不一定，假如根本沒有那麼複雜，

也許你只是需要一個 template engine 而已，

而把 React 當作 html 的 template 來用，

實在是有點太小看它了。

什麼時候該用 React 或是 React 到底好在哪裏，

這個議題其實已經超出了本篇文章的範圍，

有興趣的可以看這篇：[React Components, Elements, and Instances by Dan Abramov(Redux 作者)](https://medium.com/@dan_abramov/react-components-elements-and-instances-90800811f8ca)

這也是為什麼我一直遲遲沒有碰 Angular 的原因，

（因為我還沒遇過複雜到需要用到它的情境）

但我認為在選擇前端的框架時，這篇文章很值得一看再看：

- [界面之下：還原真實的MV*模式 ](https://github.com/livoras/blog/issues/11)

裡面並沒有太多的程式碼，只有比較 high level 的概念，

但看完你會比較理解別人說 MVC、MVP、MVVM、Model 2 是在說些什麼，

前端主要工作之一就是處理使用者介面（UI），

我認為理解這些模式是一個前端工程師必備的 common sense，

這些概念比起淘汰迅速的工具們，是比較能夠保值的，

並且也會漸漸影響你挑選工具的眼光。

而 medium 上也有許多好文章可以看，

twitter 上面也有很多大神可以讓你追蹤，

不要把這些事情當作是在大拜拜，

覺得追蹤越多人自己越屌，

重要的是你看他們生產的內容時得到了什麼。

另外臉書上的前端社團也很值得加入，台灣人的軟體能力是很強悍的：

- [Front-End Developers Taiwan](https://www.facebook.com/groups/f2e.tw/?fref=ts)

- [AngularJS.tw](https://www.facebook.com/groups/augularjs.tw/?fref=ts)

- [ReactJS.tw](https://www.facebook.com/groups/reactjs.tw/?fref=ts)

- [JavaScript.tw](https://www.facebook.com/groups/javascript.tw)

重要的是在上面發問，也會有人很熱心的回答你。

假如這樣都還是讓你資訊焦慮，可以開始訂閱一些技術週刊，

像是[碼天狗](http://weekly.codetengu.com/)、[TechBridge](http://weekly.techbridge.cc/)，

讓 curators 來幫你整理一些技術上的新知。

已經盡量精簡了資訊來源，希望能讓新手們不要太無所適從。

------

## 3. 前端工程師該懂後端嗎？

後端跟前端是完全不一樣的專業，

有人說 Node.js 能讓前端工程師跨足到後端去。

(Isomorphic？)

事實上前端工程師想往後端走還是有許多需要學習的，

不管是資料庫或是系統面，都不是平常前端會碰觸到的領域，

認為自己會寫 JavaScript 就硬上的下場通常是：

- 效能有問題

- 資安有問題

- 整個 server-side 的 code 都他媽很有問題

聽起來是很糟糕的事情，所以請千萬尊重專業，

讓我們前端歸前端、政治歸政治（欸？）。

那前端到底要理解後端到怎樣的程度呢？

這是一個很 tricky 的問題，

大部份人會說：**「至少要會接資料啦！」**

但要學到會接資料揪竟是需要怎樣的能力呢？

真的有人學到剛剛好就喊停的嗎？

最好的方法其實就是自己去玩一套網頁框架，

後端前端都寫一遍。

Rails, Laravel, Django 都是我認為不錯的選擇，

（Koa 也很不錯啦......）

重點是去感受一下自己要怎樣設計 DB 的 Schema，

怎樣做正規化、怎樣避免 N+1 Query，

以及整個框架的架構為什麼要這樣設計，

最後再跟自己拉的頁面整合在一起，然後部署上去，

（用 heroku 是有點偷懶，不過如果你對 server 真的沒興趣，還是可以考慮這樣做沒差）

等做到這一步，「至少要會接資料」這一點，

早就迎刃而解了。

對了，

記得也不要因為自己寫過後端的 code 就說自己是 full-stack，

這就跟你會收發 email 就說自己懂電腦一樣會被笑。

(IT crowd 真的是個不錯的影集)

有興趣可以看看這篇：

- [一個前端工程師眼中的 Node.js](http://www.infoq.com/cn/articles/nodejs-in-front-end-engineer-view)

可以略懂 Async 在 server 端和 client 端的差異。

----

目前大概走到這裡，還有很多沒說到，

但學個基礎開始實作後就能體會到許多了。

> 至於實作，

> 可以選擇自己寫個身體健康、參與 open source，

> 或是去實習都是非常好的選擇


不管是 RWD、mobile web、跨瀏覽器的處理、SEO，

動畫該用 CSS3 或是 JavaScript 還是 SVG？

每天都有新的問題可以鑽研，

目前就先寫到這裡啦！

希望可以改變一些覺得前端工程師只是在切切版的想法，

也希望能幫助到想往前端工程師邁進的人。