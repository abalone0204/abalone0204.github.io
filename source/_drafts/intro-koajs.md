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



# 參考資料
 
- [kick-off-koa](https://github.com/koajs/kick-off-koa)
