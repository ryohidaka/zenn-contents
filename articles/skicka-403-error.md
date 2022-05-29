---
title: "Skickaで403エラーが出た場合の対処法"
emoji: "🌟"
type: "tech"
topics: ["GCP", "Skicka", "Github Actions"]
published_at: "2022/01/22"
published: true
---

# はじめに

Skicka を用いて Github のリポジトリから GoogleDrive にデータをアップロードしようと思いました。
途中で 403 エラーに詰まってしまったので、その際の解決方法を備忘録としてを記します。

# 問題

Skicka のトークンを発行しようとしたが、途中で 403 エラーとなり進めなくなってしまった。

## 現状

下記記事を参考に実装したところ、`skicka.config`を記述する作業までは問題なく進めたが、それ以降上記問題で進めなくなった。
恐らく GCP 側で何らかの変更があったのだろう。

https://qiita.com/sekitaka_1214/items/85875d64c226b2f7ab86

上記ファイルを記述し、`skicka -no-browser-auth df`を実行すると、認証画面の URL が返却される。
指定された URL にアクセスすると、認証画面にいくはずなのだが、403 エラーとなり進むことができない。

## 原因

テストユーザに自分を登録していない場合に、403 となるらしい (Twitter 情報)

# 対応

## テストユーザを登録する

1. 「API とサービス」内の「OAuth 同意画面」にアクセスする
   ![](/images/skicka-403-error/skicka-gcp-sidemenu.png)

2. 追加ボタンを押すと入力画面が表示されるため、入力欄に自分のメールアドレスを入力し登録する。
   ![](/images/skicka-403-error/skicka-gcp-add-user.png)

## 動作確認

1. テストユーザに自分を登録した上で、再度`skicka -no-browser-auth df`を実行してみる

```shell
#skicka -no-browser-auth df
Go to the following link in your browser:
https://accounts.google.com/o/oauth2/auth?client_id=XXX.apps.googleusercontent.com...
```

2. 上記で取得した URL にアクセスする
   先ほどと違って、警告は出るものの**Continue**から先には進めそう
   ![](/images/skicka-403-error/skicka-attention-safe.png)

3. 画面の案内に従ってログイン
   ![](/images/skicka-403-error/skicka-ask-account.png)

4. トークンを取得し、入力
   ![](/images/skicka-403-error/skicka-return-token.png)

```shell
Enter verification code: { 取得したトークン }
```

4. **無事アクセス成功！！！**
   問題なく GoogleDrive の情報が取得できた！

```shell
Updating metadata cache: [=======================================================================] 100.00% 5m32s
Capacity   100.00 GiB
Trash        2.04 GiB     2.04%
Drive       12.26 GiB    12.26%
Gmail      262.72 MiB     0.26%
Photos      77.91 GiB    77.91%
Free space   7.53 GiB     7.53%
```

(100GB 契約で、空き容量が 7GB...)

5. トークン情報を取得する

```shell
# cat /root/.skicka.tokencache.json
{"ClientId":"{client_id}","access_token":"{token}","token_type":"Bearer","refresh_token":"{token}","expiry":"2022-01-22T14:23:59.9556225Z"}#
```

あとは、このトークン情報を用いて実装するだけです。

# 参考文献

[テストユーザが必要という情報が載っていたツイート](https://twitter.com/mushroom080/status/1403796268644585472?s=20)

https://qiita.com/satackey/items/34c7fc5bf77bd2f5c633

https://qiita.com/sekitaka_1214/items/19b27a837fe87b7b9ff9

https://qiita.com/satackey/items/34c7fc5bf77bd2f5c633

https://qiita.com/sekitaka_1214/items/85875d64c226b2f7ab86

https://ifritjp.github.io/blog2/public/posts/2020/2020-12-05-lns-release/
