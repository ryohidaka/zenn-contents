---
title: "Instafeed.jsで案内されているトークンジェネレータの代替サービス"
emoji: "🔑"
type: "tech"
topics: ["instagram", "instafeed"]
published_at: "2022/01/20"
published: true
---

## 問題

- [Instafeed.js](https://instafeedjs.com/#/)というサービスを用いて、インスタグラムの画像を自身の Web ページに掲載しようとしたところ、案内されている Heroku のパッケージ[Instagram Token Agent](https://github.com/companionstudio/instagram-token-agent)が**一部機能有料化**のため、使用できなくなっていました。

※[issue](https://github.com/companionstudio/instagram-token-agent/issues/33)にも上っていた

## 対応方法

- [Instant Tokens](https://www.instant-tokens.com/home)というサービスでトークン取得処理を代替する

## 手順

### 1. Instagram 基本表示 API の設定

- [API のドキュメント](https://developers.facebook.com/docs/instagram-basic-display-api/getting-started)の手順に従ってステップ 6 までの工程を行い、**アクセストークンが取得できるように**する

### 2. Instant Tokens とインスタグラムアカウントを連携する

1. [Instant Tokens](https://www.instant-tokens.com/home)にアクセスし、アカウント登録を行う
   ![Instant Tokens](/images/instafeed-replacement-token-generator/instant-tokens-login.png)

2. インスタグラムアカウントを連携する
   ![](/images/instafeed-replacement-token-generator/instant-tokens-register.png)

3. 設定を開き、JS ファイルの URL もしくは リクエスト先 URL を取得する
   ![](/images/instafeed-replacement-token-generator/instant-tokens-list.png)

4. Instafeed とトークン取得用 JS ファイルをインポートする

```html:index.html
<!-- Instafeed -->
<script type="text/javascript" src="path/to/instafeed.min.js"></script>

<!-- token -->
<script type="text/javascript" src="https://ig.instant-tokens.com/users/{user_id}/instagram/{app_id}/token.js?userSecret={user_secret}"></script>
```

5. 変数名`InstagramToken`にトークンが代入されているため、これを用いて表示する

```js: token
var InstagramToken = {access_token};
```

```html:index.html
<div id="instafeed"></div>

<script type="text/javascript">
    var feed = new Instafeed({
      accessToken: InstagramToken
    });
    feed.run();
</script>
```
