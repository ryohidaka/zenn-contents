---
title: "LaravelでBacked Enumを用いた際に、valueからcaseを参照する方法"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["php", "Laravel", "enum"]
published: false
---

## 概要

Laravel で Backed Enum を用いた際に個人的に詰まったポイントがあったので、まとめました。

### 目的

Laravel で Backed Enum に定義した`value`から`label`を参照する

#### 例

- `SuitID`が**1**の`case`を参照する
- 参照した`case`に対して`label`メソッドを用いてラベル名を取得する。
- 下記 Enum の場合、**ハート**が返却されること

```php:app/Enums/Suit.php
<?php

namespace App\Enums

enum Suit
{
    case Hearts = 1;
    case Diamonds = 2;
    case Clubs = 3;
    case Spades = 4;

    public function label(): string
    {
      return match ($this) {
        Suit::Hearts => 'ハート',
        Suit::Diamonds => 'ダイヤモンド',
        Suit::Clubs => 'クラブ',
        Suit::Spades => 'スペード',
      }
    }
}
```

### 結論

`from`メソッドの引数に`value`を指定することで、取得できる。

```php
use App\Enum\Suit;

// 取得したいSuitのvalueを指定する
$suit_value = 1;

// valueから該当のSuitを取得し、labelメソッドでラベル名を取得する
$suit_label = Suit::from($suit_value)->label();
```

## PHP8.1 から追加された Enum について

Enum 型を使えば、簡単に 複数の定数を定義できます。

### 定義例

```php:app/Enums/Suit.php
<?php

namespace App\Enums

enum Suit
{
    case Hearts = 1;
    case Diamonds = 2;
    case Clubs = 3;
    case Spades = 4;

    public function label(): string
    {
      return match ($this) {
        Suit::Hearts => 'ハート',
        Suit::Diamonds => 'ダイヤモンド',
        Suit::Clubs => 'クラブ',
        Suit::Spades => 'スペード',
      }
    }
}
```

### 参照方法

定義した内容は、以下のように`Enumのクラス名`+`変数名`+`value`で参照できます。

```php:routes/web.php
<?php

use App\Enum\Suit;

Route::get('/', function () {

  var_dump(Suit::Hearts->value)
  // 1
});
```

また、下記のように新たにメソッドを定義することもできます。

### label メソッドの例

- 日本語名を返却する`labelメソッド`を追加しました。

```php:app/Enums/Suit.php
<?php

namespace App\Enums

enum Suit
{
    case Hearts = 1;
    case Diamonds = 2;
    case Clubs = 3;
    case Spades = 4;

    public function label(): string
    {
      return match ($this) {
        Suit::Hearts => 'ハート',
        Suit::Diamonds => 'ダイヤモンド',
        Suit::Clubs => 'クラブ',
        Suit::Spades => 'スペード',
      }
    }
}
```

定義した内容は、以下のように`Enumのクラス名`+`変数名`+`メソッド名`で参照できます。

```php:routes/web.php
<?php

use App\Enum\Suit;

Route::get('/', function () {

  var_dump(Suit::Hearts->label())
  // 1
});
```

