---
title: "Zaim APIとGoを用いて、ZaimのデータをCSV出力する"
emoji: "🐥"
type: "tech"
topics: ["Go", "Zaim", "CSV", "API"]
published: false
---

# 目的

- Zaim のデータを CSV に出力する仕組みを作る
- バックアップデータは、Zaim のファイル出力機能で出力されるものと同じ内容にする(カラム名を揃える)

# 手順

## API 登録・トークン取得

API 登録、トークン取得は[以前の記事](https://zenn.dev/hidaka/articles/zaim-backup-python)を参考にしてください。

## コードを実装

#### Zaim のデータを取得・操作するパッケージを追加する

```shell:ターミナル
go get github.com/s-sasaki-0529/go-zaim
```

https://github.com/s-sasaki-0529/go-zaim

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

トークン情報を読み込む`config.go`とデータを取得する`zaim.go`、各処理を呼び出す`main.go`を実装します。

```go:config.go
package main

import (
	"fmt"
	"os"

	gozaim "github.com/s-sasaki-0529/go-zaim"

	"github.com/joho/godotenv"
)

func GetClient() *gozaim.Client {
	err := godotenv.Load(".env")

	if err != nil {
		fmt.Printf("読み込み出来ませんでした: %v", err)
	}

	c := gozaim.NewClient(
		os.Getenv("CONSUMER_KEY"),
		os.Getenv("CONSUMER_SECRET"),
		os.Getenv("ACCESS_TOKEN"),
		os.Getenv("ACCESS_SECRET"),
	)

	return c
}
```

```go:zaim.go
package main

import (
	"fmt"
	"net/url"

	gozaim "github.com/s-sasaki-0529/go-zaim"
)

type ZaimData struct {
	money      []gozaim.Money
	categories []gozaim.Category
	genres     []gozaim.Genre
	accounts   []gozaim.Account
}

// Zaimのデータを取得する
func GetZaimData(c *gozaim.Client) ZaimData {

	// データ一覧の取得
	m, err := c.FetchMoney(url.Values{})
	if err != nil {
		fmt.Println("Failed to get money", err)
	}

	msg := fmt.Sprintf("%d 件のデータを取得しました。\n", len(m))
	fmt.Println(msg)

	// カテゴリ一覧取得
	ca, err := c.FetchCategories()
	if err != nil {
		fmt.Println("Failed to get categories", err)
	}

	// ジャンル一覧取得
	g, err := c.FetchGenres()
	if err != nil {
		fmt.Println("Failed to get genres", err)
	}

	// 口座一覧取得
	a, err := c.FetchAccounts()
	if err != nil {
		fmt.Println("Failed to get accounts", err)
	}

	result := ZaimData{
		m,
		ca,
		g,
		a,
	}

	return result
}
```

```go:main.go
package main

import "fmt"

func main() {

	// クライアント設定
	c := GetClient()

	// データ取得
	d := GetZaimData(c)
	fmt.Println(d)

}
```

## まとめ

## 参考文献
