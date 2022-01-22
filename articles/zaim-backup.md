---
title: "Zaim APIとPythonを用いて、ZaimのデータをCSV出力する"
emoji: "🐥"
type: "tech"
topics: ["Python", "Zaim", "CSV"]
published: false
---

# 目的

- Zaim のデータを CSV に出力する仕組みを作る
- バックアップデータは、Zaim のファイル出力機能で出力されるものと同じ内容にする(カラム名を揃える)

# 手順

## API 登録

### Zaim developers に登録

https://dev.zaim.net/

### アプリケーションを登録

アプリケーションを登録する([登録画面](https://dev.zaim.net/home/clients/add))

- 名称: 任意の名称
- サービス種: クライアントアプリ
- 概要: 任意の概要
- 組織: 任意の組織名
- サービスの URL: 任意の URL
- アクセスレベル

  - 読み込み: 必要
  - 書き込み: 不要
  - 永続許可: 必要

![アプリケーションを登録](/images/zaim-backup/zaim-dev-register.png)

### 認証情報を確認する

1. [登録済みアプリケーション](https://dev.zaim.net/home/clients)の一覧から作成したアプリケーションを選択し、詳細表示する
   ![登録済みアプリケーション](/images/zaim-backup/zaim-dev-list.png)
2. 詳細画面の**コンシューマ ID**と**コンシューマシークレット**を控える
   ![詳細表示](/images/zaim-backup/zaim-dev-info.png)

これで API 登録は完了です。

## まとめ

## 参考文献
