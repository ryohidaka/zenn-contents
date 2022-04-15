---
title: "LaravelでvalueからEnumを参照する方法"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["php", "laravel", "enum"]
published: false
---

## 概要

### 目的

`value`から`label`を参照する

#### 例

- `SuitID`が**1**の`Suit`の名称を取得する
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

