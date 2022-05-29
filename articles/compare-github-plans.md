---
title: "Githubの無料プラン(Free)と有料プラン(Pro)を比較"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHub]
published_at: "2022/05/01"
published: true
---

## はじめに

Github の個人向け無料プラン(`Free`)と有料プラン(`Pro`)の機能を比較しまとめました。

## おことわり

- [GitHub のアップグレードページ](https://github.com/settings/billing/plans)を参考に作成しています。
- 記載している情報は 2022 年 5 月現在のものです。
- Free, Pro 共に使用不可な機能に関しては記載していません。
- Team や Enterprise などの比較は[こちら](https://github.co.jp/pricing.html)を参考にしてください。

## 比較

### コード管理

#### パブリックリポジトリ

- Free, Pro ともに数の制限なく作成できます。

#### プライベートリポジトリ

- Free, Pro ともに数の制限なく作成できます。

### ワークフロー

#### GitHub Actions

- **プライベートリポジトリ**で使用する際の利用可能な実行時間に違いがあります。
- パブリックリポジトリで使用する分には制限がありません。

|                        | Free         | Pro          |
| ---------------------- | ------------ | ------------ |
| プライベートリポジトリ | 2000 分 / 月 | 3000 分 / 月 |
| パブリックリポジトリ   | 無制限       | 無制限       |

#### GitHub Packages

- 作成したプロダクトをコミュニティ標準のパッケージマネージャー(`npm`や`nuget`)から公開できる機能です。
- Free と Pro では以下の違いがあります。
  - 公開可能な容量
  - GitHub Actions 以外を経由した際のデータ転送量
- パブリックリポジトリには制限はありません。

|                | Free     | Pro       |
| -------------- | -------- | --------- |
| 公開可能な容量 | 500MB    | 2GB       |
| データ転送量   | 1GB / 月 | 10GB / 月 |

https://github.co.jp/features/packages

#### Protected Branches

- 特定のブランチに対してコミットルールを設定する機能です。
- Free の場合は、**パブリックリポジトリのみ**設定可能です。

| Free                     | Pro    |
| ------------------------ | ------ |
| パブリックリポジトリのみ | 無制限 |

https://docs.github.com/ja/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches

#### Code Owners

- プルリクエストが作成された際に、自動的にレビュワーが指定されるように定義できる機能です。
- Free の場合は、**パブリックリポジトリのみ**設定可能です。

| Free                     | Pro    |
| ------------------------ | ------ |
| パブリックリポジトリのみ | 無制限 |

https://docs.github.com/ja/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners

#### Multiple Pull Request Assignees

- プルリクエストのレビュイー(もしくは担当者)を複数指定できる機能です。
- Free の場合は、**パブリックリポジトリのみ**設定可能です。

| Free                     | Pro    |
| ------------------------ | ------ |
| パブリックリポジトリのみ | 無制限 |

https://docs.github.com/ja/issues/tracking-your-work-with-issues/assigning-issues-and-pull-requests-to-other-github-users

#### Multiple Pull Request Reviewers

- プルリクエストのレビュワーを複数指定できる機能です。
- Free の場合は、**パブリックリポジトリのみ**設定可能です。

| Free                     | Pro    |
| ------------------------ | ------ |
| パブリックリポジトリのみ | 無制限 |

https://docs.github.com/ja/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/requesting-a-pull-request-review

### コラボレーション

#### Pages and Wikis

- GitHub Pages: 静的ページを公開する機能です。
- Wiki: Wiki ページを作成する機能です。
- Free の場合は、**パブリックリポジトリのみ**設定可能です。

| Free                     | Pro    |
| ------------------------ | ------ |
| パブリックリポジトリのみ | 無制限 |

https://pages.github.com/

https://docs.github.com/ja/communities/documenting-your-project-with-wikis/adding-or-editing-wiki-pages

## まとめ

調べた結果、自分は以下の理由で無料プランにダウングレードすることにしました。

- GitHub Pages の使用頻度が下がったため
  - [Netlify](https://www.netlify.com/) や [Vercel](https://vercel.com/) などのホスティングサービスを使うようになった
- プライベートリポジトリを複数人で使用する予定がないため
- GitHub Actions を使っているが、制限に達するほどの使用予定がないため

個人的には無料プランでプライベートリポジトリが作成できるようになったのが、一番感激しています。
(学生の頃の恥ずかしいコードを隠せるので...)

皆さんのプラン選びの参考になれば幸いです。
