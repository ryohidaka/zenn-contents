---
title: "Next.jsのAppRouter機能を用いた静的サイトの作成"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs"]
published: true
---

[Next.js 13.3](https://nextjs.org/blog/next-13-3)から`App Router`機能が静的エクスポート(`next export`)に対応しました。

![静的エクスポートに対応](/images/nextjs-app-router-blog/static-export-note.png)

こちらは、以下のスクラップをまとめたものです。

https://zenn.dev/hidaka/scraps/32ba6e601fe460

## TL;DR
- `create-next-app`を用いてアプリケーションのテンプレートを作成
- 静的出力のオプションを有効化
- pagesディレクトリとAPI Routeを削除
- ビルド

## Next.jsのアプリケーションを作成

### アプリケーションテンプレートを作成

`create-next-app`を用いてアプリケーションのテンプレートを作成

```zsh:terminal
$ npx create-next-app@latest        
```

`app/`ディレクトリのオプションを有効化する

```zsh:terminal                                       
Need to install the following packages:
  create-next-app@13.3.0
Ok to proceed? (y) y
✔ What is your project named? … nextjs-demo
✔ Would you like to use TypeScript with this project? … No / Yes
✔ Would you like to use ESLint with this project? … No / Yes
✔ Would you like to use Tailwind CSS with this project? … No / Yes
✔ Would you like to use `src/` directory with this project? … No / Yes
✔ Would you like to use experimental `app/` directory with this project? … No / Yes
✔ What import alias would you like configured? … @/*
Creating a new Next.js app in /Users/ryohidaka/nextjs-demo.
```

### 静的出力のオプションを有効化

[ドキュメント](https://beta.nextjs.org/docs/configuring/static-export#configuration)に倣って`next.config.js`を修正。

:::message
ドキュメントには`appDir: true`の記述はないが、含めないとエラーが発生するので注意

> Error: > The `app` directory is experimental. To enable, add `appDir: true` to your `next.config.js` configuration under `experimental`. See https://nextjs.org/docs/messages/experimental-app-dir-config

:::

```js:next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  // @see https://beta.nextjs.org/docs/configuring/static-export#configuration
  output: "export",
};

module.exports = nextConfig
```

### pagesディレクトリとAPI Routeを削除

初期生成ファイルに`pages/api/hello.ts`が含まれてているため削除する。

> error - API Routes cannot be used with "output: export". See more info here: https://nextjs.org/docs/advanced-features/static-html-export

また、`pages`ディレクトリが存在する場合、`app`ディレクトリと競合して`404`エラーになるため、`pages`ディレクトリごと削除する。


## 動作確認

### ホットリロードで実行できるか確認

`next dev`を実行し、Next.jsのデフォルトページが表示できることを確認

![Next.jsのデフォルトページ](/images/nextjs-app-router-blog/nextjs-default-page.png)

## ビルド

静的出力を行う。

> Deploying
> With a static export, Next.js can be deployed and hosted on any web server that can serve HTML/CSS/JS > static assets.
> 
> When running next build, Next.js generates the static export into the out folder. Using next export is no longer needed. For example, let's say you have the following > routes:

以前は`next build && next export`のようにビルドした上で静的出力する対応が必要だったが、Next.js13.3からは`next build`だけでoutディレクトリに出力してくれるようになったらしい。

https://beta.nextjs.org/docs/configuring/static-export#deploying

無事出力処理が行われ、`out`ディレクトリ以下に展開されたることを確認

![ビルド成功時のコンソール出力](/images/nextjs-app-router-blog/out-console.png)

![ビルド成功時のoutディレクトリ](/images/nextjs-app-router-blog/out-dir.png)