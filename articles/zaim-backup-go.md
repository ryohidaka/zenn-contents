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

データが取得できましたが、カテゴリや口座等は ID しか記載されていません。
そのため、ID をもとに各種名称を付与する処理を実装します。

#### ID をもとに、名称を付与する

名称を付与する処理を追記します。

```go:zaim.go
type MoneyJP struct {
	Date         string `csv:"日付"`
	Mode         string `csv:"方法"`
	Category     string `csv:"カテゴリ"`
	Genre        string `csv:"カテゴリの内訳"`
	From         string `csv:"支払元"`
	To           string `csv:"入金先"`
	Name         string `csv:"品目"`
	Comment      string `csv:"メモ"`
	Place        string `csv:"お店"`
	CurrencyCode string `csv:"通貨"`
	Income       int    `csv:"収入"`
	Payment      int    `csv:"支出"`
	Transfer     int    `csv:"振替"`
}

// 種別IDをもとに、種別名を付与する
func ConvertData(datas ZaimData) []MoneyJP {
	var money []MoneyJP

	for _, v := range datas.money {
		p, i, t := GetAmount(v.Mode, v.Amount)
		cm := MoneyJP{
			v.Date,
			v.Mode,
			GetCategoryName(v.CategoryID, datas.categories),
			GetGenreName(v.GenreID, datas.genres),
			GetAccountName(v.FromAccountID, datas.accounts),
			GetAccountName(v.ToAccountID, datas.accounts),
			v.Name,
			v.Comment,
			v.Place,
			v.CurrencyCode,
			p,
			i,
			t,
		}

		money = append(money, cm)

	}
	return money
}

// IDに紐づくカテゴリ名を返却
func GetCategoryName(id int, categories []gozaim.Category) string {
	for _, v := range categories {
		if v.ID == id {
			return v.Name
		}
	}
	return ""
}

// IDに紐づくジャンル名を返却
func GetGenreName(id int, genres []gozaim.Genre) string {
	for _, v := range genres {
		if v.ID == id {
			return v.Name
		}
	}
	return ""
}

// IDに紐づく口座名を返却
func GetAccountName(id int, accounts []gozaim.Account) string {
	for _, v := range accounts {
		if v.ID == id {
			return v.Name
		}
	}
	return "-"
}

// 方法に応じた額を返却
func GetAmount(mode string, amount int) (int, int, int) {
	var p, i, t int

	switch mode {
	case "payment":
		p = amount
	case "income":
		i = amount
	case "transfer":
		t = amount
	default:
	}

	return p, i, t

}
```

```go:main.go
// 種別名を付与する
cd := ConvertData(d)
fmt.Println(cd)
```

続いてカラム名を日本語に変更し、CSV 出力する処理を実装します。

#### データを出力するコードを追加

カラム名を日本語に変更し、ユーザ ID など不要なデータを除去します。
続いて、整頓されたデータを CSV 出力します。

```go:zaim.go
// CSV出力
func OutputCSV(money []MoneyJP) {
	file, _ := os.OpenFile("zaim-backup.csv", os.O_RDWR|os.O_CREATE, os.ModePerm)
	defer file.Close()

	// csvファイルを書き出し
	gocsv.MarshalFile(&money, file)

}
```

```go:main.go
// CSV出力する
OutputCSV(cd)
```

これでコードの実装は完了です。

## まとめ

## 参考文献
