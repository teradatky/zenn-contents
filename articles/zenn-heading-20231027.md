---
title: "Zenn の見出しレベルはどこから始めるか"
emoji: "📝"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Zenn", "Markdown", "HTML"]
published: true
---

## 結論

見出しは `見出し2` `H2` `##` から始めましょう。
ただし意図があれば `見出し1` `H1` `#` も可です。

## 背景

見出しレベルは `H1` から存在しますが、そのまま使ってよいのか気になったので調べました。

## 公式では

以下の通り `H2` から始めることを推奨しています。

:::message
アクセシビリティの観点から `見出し2` から始めることをおすすめします
:::

https://zenn.dev/zenn/articles/markdown-guide

## Linter では

Markdown の Linter は `H1` が複数あると警告してきます。

![warning](/images/zenn-heading-20231027/heading1.png)

## 実際のページ表示は

`H1` で記載した見出しは `H1` として表示されます。

![heading1](/images/zenn-heading-20231027/heading2.png =600x)
![heading1](/images/zenn-heading-20231027/heading3.png =600x)

タイトルも `H1` として表示されます。

![title](/images/zenn-heading-20231027/heading4.png =600x)
![title](/images/zenn-heading-20231027/heading5.png =600x)

このため `H1` から始めると、そのページには必ず `H1` が複数存在することになります。

## 複数の `H1` は悪なのか

`H1` を複数使ってはいけないというルール、不利に扱うというアルゴリズムが明記されたサービスは見つかりませんでした。
そのため文字サイズやスペースが `H1` の方が適していると判断した場合は利用しても構わないでしょう。

![Google Search](/images/zenn-heading-20231027/heading6.png)

## `H2` から始めた場合の目次

Zenn が自動作成する目次に影響がないか確認しましたが、問題ありませんでした。

### `H1` から始めると `H1` と `H2` が目次になる

![h1h2](/images/zenn-heading-20231027/heading7.png =600x)
![h1h2](/images/zenn-heading-20231027/heading8.png =600x)

### `H2` から始めると `H2` と `H3` が目次になる

![h2h3](/images/zenn-heading-20231027/heading9.png =600x)
![h2h3](/images/zenn-heading-20231027/heading10.png =600x)

以上から、特にこだわりがなければ `H2` から始め、必要に応じて `H1` を利用することをオススメします。
