{{{
"title": "自動化發布",
"date": "2016/1/6",
"tags": ["auto-realease", "JavaScript", "CI"]
}}}

影片的話可以參照 egghead.io 上面的免費課程

這裏是根據裡面的 practice 實作後的紀錄。


- Test.

- mocha, chai

- 先測試


- Semantic release

```bash
> semainti-release-cli setup

? Is the GitHub repository private? No
? What is your npm registry? https://registry.npmjs.org/
? What is your npm username? tjku
? What is your npm email? abalone0204@gmail.com
? What is your GitHub username? abalone0204
? What CI are you using? Travis CI
? What kind of `.travis.yml` do you want? Single Node.js version.
```

接著會新增數個檔案和變更 package.json 以及新增 travis.yml

package.json 中的 version 被拿掉了，

如果我們要用 npm install 東西的話，

這樣會噴錯。

所以我們還是要手動將其加回去

```json
{
    ...
    "version": "0.0.0-semantically-released",
    ...
}
```

在 travis.yml 裡面設定要通過 test 才能 release 新版本

```yml
script:
  - npm run test
```

- Conventional commit


- Automatically Releasing with TravisCI

- Automatically running tests before commits with ghooks


- Goal
    
    - 在 commit 之前先跑測試

    - 在 push 前可以先 bundle

- commitizen和 ghooks的 config 推薦寫法，影片中的樣式已經舊了

```json
"config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog/"
    },
    "ghooks": {
      "pre-commit": "npm run test:single"
    }
}
```

# 測試覆蓋率

使用 istanbul


```json
{
    "test:single": "istanbul cover -x *.test.js _mocha -- -R $(find test -name *.test.js)"
}
```

- `-x`是因為我們不需要去測試測試用的js

- `_mocha` 則是為搭配`istanbul`所使用的