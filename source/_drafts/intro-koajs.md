---
title: Koa Kick off
tags: [koajs]
---

Koajs

# routing

koa 沒有自帶的 routing

所以可以自己寫 或是用別人的 middleware

假如要略過這個 middleware 往下一個去的話

```js
app.use(function*(next) {
    if (skip) {
        return yield next
    }
})
```

`skip`在這裡是條件判斷式的結果。

# request body

> 對`/` 執行 POST method
> body: `{name: 'string'}
> respond 要把 name 對應的 value 轉成大寫

# Response body

  * Strings
  * Buffers
  * Streams (node)
  * JSON Objects

# Content Header

    - Content-Type
    - Content-Length
    - Content-Encoding

要知道 request 的 type 比較困難，

koa 有個好用的 helper function `is`，

讓我們不需要去自己寫 regular expression
 
```js
this.request.is('image/*') // => image/png
this.request.is('text') // => text or false
```
> 當然 response 也有一樣的 `is`

可以在 this.body 直接放入 js 的 object

就會預設回傳 json 了（連 header 都不用設定）

# MIDDLEWARE

>  * responseTime: record each request's response time(ms), set the response header `X-Response-Time`.
> * upperCase: convert response body to upper case.

hint: middleware 也只是個會 return  generator function 的 function

In koa middlewares, use this.set(name, val) to set a response header.
And change response body by re

# Cookie

練習用 cookie 來存 user's view time

- [express: cookies](https://github.com/expressjs/cookies)

這裡的 cookie 還是用原本 express 上面的實作，

文件可以看那裡的

# Session

```js
app.keys = ['secret', 'keys'];
app.use(session(app));
// Then I can use `this.session` in hanlder
```

# Template

# 參考資料
 
- [kick-off-koa](https://github.com/koajs/kick-off-koa)

