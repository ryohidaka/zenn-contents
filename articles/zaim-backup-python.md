---
title: "Zaim APIとPythonを用いて、ZaimのデータをCSV出力する"
emoji: "🐥"
type: "tech"
topics: ["Python", "Zaim", "CSV", "API"]
published_at: "2022/01/22"
published: true
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
続いて Python でデータを取得するコードを実装します。

## コードを実装

### トークンを取得

API を呼び出すには先ほど控えた認証情報に加えて、**アクセストークン**と**アクセスシークレット**が必要なので、それらを取得するコードを実装します。

#### Zaim のデータを取得・操作する Python パッケージを追加する

```shell:ターミナル
pip install pyzaim
```

https://pypi.org/project/pyzaim/

#### トークンを取得するためのコードを追加する

```python:auth.py
from pyzaim import get_access_token

get_access_token()
```

#### トークン取得用コードを実行し、指示に従って認証情報を入力する

1. コンシューマ ID とコンシューマシークレットを聞かれるので入力
1. 認証ページの URL が表示されるので、アクセスして許可
1. 遷移先ページのソースコードから「oauth_verifier」と書いてあるコードをコピーして入力
1. 問題なければアクセストークンとアクセスシークレットが表示される

```shell:ターミナル
% python auth.py
Please input consumer ID : { consumer-key }
Please input consumer secret : { consumer-secret }


Please go here and authorize :  https://auth.zaim.net/users/auth?oauth_token={ oauth-token }
Please input oauth verifier : { oauth-token }


access token : xxxxx
access token secret : xxxxx
oauth verifier : xxxxx,xxxxx
```

続いて、取得したトークン情報を用いてデータを取得するコードを実装します。

### データを取得するコードを追加する

#### env ファイルに取得したトークン情報を記載する

`.env`ファイルを作成し、取得したトークンを記載します。

```shell:.env
CONSUMER_KEY=
CONSUMER_SECRET=
ACCESS_TOKEN=
ACCESS_SECRET=
```

#### データを取得するコードを追加する

トークン情報を読み込む`config.py`とデータを取得する`zaim.py`、各処理を呼び出す`main.py`を実装します。

```python:config.py
# .env ファイルをロードして環境変数へ反映
from dotenv import load_dotenv
load_dotenv()

# 環境変数を参照
import os
CONSUMER_KEY = os.getenv('CONSUMER_KEY')
CONSUMER_SECRET = os.getenv('CONSUMER_SECRET')
ACCESS_TOKEN = os.getenv('ACCESS_TOKEN')
ACCESS_SECRET = os.getenv('ACCESS_SECRET')

```

```python:zaim.py
import config
from pyzaim import ZaimAPI

def getZaimData():
    """Zaimのデータを取得する
    """

    consumer_key = config.CONSUMER_KEY
    consumer_secret = config.CONSUMER_SECRET
    access_token = config.ACCESS_TOKEN
    access_secret = config.ACCESS_SECRET

    api = ZaimAPI(consumer_key, consumer_secret,
                  access_token, access_secret, 'verifier')
    # データ一覧の取得
    datas = api.get_data()

    # カテゴリ一覧情報
    categories = api.category_itos

    # ジャンル一覧情報を取得
    genres = api.genre_itos

    # 口座一覧情報を取得
    accounts = api.account_itos

    msg = str(len(datas)) + " 件のデータを取得しました"
    print(msg)

    return [datas, categories, genres, accounts]
```

```python:main.py
import zaim

# データ取得
datas, categories, genres, accounts = zaim.getZaimData()
```

データが取得できましたが、カテゴリや口座等は ID しか記載されていません。
そのため、ID をもとに各種名称を付与する処理を実装します。

#### ID をもとに、名称を付与する

名称を付与する処理を追記します。

```python:zaim.py
def convertData(datas, categories, genres, accounts):
    """IDをもとに名称を付与する
    """

    for data in datas:

        # カテゴリ名を付与
        categoryId = int(data["category_id"])
        data["category"] = categories[categoryId] if categoryId > 0 else ""

        # 内訳名を付与
        genreId = int(data["genre_id"])
        data["genre"] = genres[genreId] if genreId > 0 else ""

        # 口座名を付与
        fromAccountId = int(data["from_account_id"])
        data["from"] = accounts[fromAccountId] if fromAccountId > 0 else "-"

        toAccountId = int(data["to_account_id"])
        data["to"] = accounts[toAccountId] if toAccountId > 0 else "-"

    return datas
```

```python:main.py
# 種別名を付与する
datas = zaim.convertData(datas, categories, genres, accounts)
```

続いてカラム名を日本語に変更し、CSV 出力する処理を実装します。

#### データを出力するコードを追加

カラム名を日本語に変更し、ユーザ ID など不要なデータを除去します。
続いて、整頓されたデータを CSV 出力します。

```python:zaim.py
import csv

def outputCSV(datas):
    """カラム名を日本語に置換し、CSV出力する
    """

    # 種別名を日本語に置換
    for data in datas:
        keys = {
            "date": "日付",
            "mode": "方法",
            "category": "カテゴリ",
            "genre": "カテゴリの内訳",
            "from": "支払元",
            "to": "入金先",
            "name": "品目",
            "comment": "メモ",
            "place": "お店",
            "currency_code": "通貨"
        }
        for k, v in keys.items():
            data[v] = data.pop(k)

        # 入出金
        data["収入"] = data.pop("amount") if data["方法"] == "income" else 0
        data["支出"] = data.pop("amount") if data["方法"] == "payment" else 0
        data["振替"] = data.pop("amount") if data["方法"] == "transfer" else 0

        # 不要なキーを削除
        unUsedKeys = ["id", "user_id", "category_id",
                      "genre_id", "from_account_id", "to_account_id", "active", "created", "receipt_id", "place_uid", "original_money_ids"]
        for key in unUsedKeys:
            if(key in data):
                data.pop(key)

    # ヘッダーを指定
    fieldName = list(keys.values())
    fieldName.extend(['収入', '支出', '振替'])

    # CSV出力
    with open('zaim-backup.csv', 'w', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldName)
        writer.writeheader()
        writer.writerows(datas)
```

```python:main.py
# CSV出力
zaim.outputCSV(datas)
```

これでコードの実装は完了です。

### 実装したコードを実行する

ターミナルなどシェルスクリプトが実行できるアプリケーションで、下記コマンドを実行します。

```shell:ターミナル
% python main.py
```

コンソールに取得件数が表示され、`zaim-backup.csv`というファイルが生成されます。

## まとめ

今回実装したコードは下記リポジトリで公開しています。

https://github.com/ryohidaka/zaim-backup

## 参考文献

https://maku77.github.io/python/env/dotenv.html

https://laboratory.kazuuu.net/saving-a-dictionary-to-a-csv-file-in-python/#toc5
