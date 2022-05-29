---
title: "Gatsbyで作成したページにインスタグラムの画像を表示 ( 非公開アカウント対応)"
emoji: "📸"
type: "tech"
topics: ["gatsby"]
published_at: "2022/01/20"
published: true
---

## 概要

Gatsby で作成したページにインスタグラムの画像を表示しようとした際の備忘録です。

## 環境

- gatsby: 4.4.0

## 手順

### パッケージを追加

- [gatsby-source-instagram-all](https://www.gatsbyjs.com/plugins/gatsby-source-instagram-all/)を追加

```shell:terminal
npm i gatsby-source-instagram-all
```

### プラグイン設定を変更

- `gatsby-config.js`に設定を追記する

```js:gatsby-config.js
plugins: [
    `gatsby-source-instagram-all`,
]
```

### Instant Tokens に登録する

- こちらの記事を参考にアカウント作成を行う

https://zenn.dev/hidaka/articles/4d34f6dccf296a

### アクセストークンを取得し、プラグインに渡す

1. .env にインスタグラムの情報を記述する

```shell:.env
INSTAGRAM_USER_ID=
INSTAGRAM_APP_ID=
INSTAGRAM_APP_SECRET=
```

2. axios を用いて Instant Tokens にリクエストを行い、返却されたアクセストークンをプラグインの設定に追加する

```js:gatsby-node
const axios = require("axios")
require("dotenv").config()

exports.onPreInit = async ({ actions, store }) => {
  const { setPluginStatus } = actions
  const state = store.getState()
  const plugin = state.flattenedPlugins.find(
    plugin => plugin.name === "gatsby-source-instagram-all"
  )
  if (plugin) {
    const userId = process.env.INSTAGRAM_USER_ID
    const appId = process.env.INSTAGRAM_APP_ID
    const userSecret = process.env.INSTAGRAM_APP_SECRET

    const instagramToken = await axios({
      method: "GET",
      url: `https://ig.instant-tokens.com/users/${userId}/instagram/${appId}/token?userSecret=${userSecret}`,
    }).then(response => {
      return response.data.Token
    })
    plugin.pluginOptions = {
      ...plugin.pluginOptions,
      ...{ access_token: instagramToken || "" },
    }
    setPluginStatus({ pluginOptions: plugin.pluginOptions }, plugin)
  }
}
```

3. GraphQL クエリを記述し、インスタグラム画像のリストを取得する

```js
  const result = await graphql(
    `
      query {
        allInstagramContent() {
          nodes {
            id
            media_url
            caption
            album {
              id
              media_url
            }
            timestamp(formatString: "DD MMM, YYYY")
          }
        }
      }
    `
```
