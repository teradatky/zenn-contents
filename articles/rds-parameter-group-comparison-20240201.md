---
title: "RDS のエンジンバージョンアップグレード時にはパラメータグループを確認しよう！"
emoji: "🐘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "RDS", "DB"]
published: true
---

## 概要

RDS を運用していると、エンジンバージョンのアップグレードが必要になります。単にエンジンバージョンを変更するだけでは問題が起きる可能性があるため、パラメータグループを確認しましょう。今回はその確認に便利なパラメータグループの比較機能を紹介します。

![overview](/images/rds-parameter-group-comparison-20240201/overview.png)

### パラメータグループとは

パラメータグループとは、DB インスタンスやクラスターに適用されるエンジンの設定値の集合です。RDS の作成時に特に設定していなかった、という場合はデフォルトのパラメータグループが割り当てられているはずです。DB のパラメータは動作やパフォーマンスのチューニングに重要な要素です。

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/parameter-groups-overview.html

### なぜパラメータグループを確認するのか

- アプリケーションの動作やパフォーマンスに影響を及ぼさないため
- パラメータグループ自体や静的パラメータの変更には DB の再起動が必要になるため

## パラメータグループ比較

パラメータグループには数百のパラメータがあり、手動でひとつずつ確認していくのは大変です。パラメータグループの比較を活用していきましょう。

### 既存のパラメータグループとデフォルトのパラメータグループを比較

まずは現在のパラメータグループがデフォルト値とどのような差分があるか確認しましょう。
デフォルトのパラメータグループは RDS での利用状況で存在有無が変わるため、まずデフォルト値のパラメータグループを作成します。

![create_parameter_group1](/images/rds-parameter-group-comparison-20240201/create_parameter_group1.png)
![create_parameter_group2](/images/rds-parameter-group-comparison-20240201/create_parameter_group2.png)

作成できたらパラメータグループ一覧画面に戻ります。2 つのパラメータグループを選択し、アクション＞比較をクリックすることで、既存のパラメータグループに対する意図した設定変更を調べることができます。

:::message
DB インスタンスパラメータグループと DB クラスターパラメータグループは比較することができません
:::

![compare_parameter_group1](/images/rds-parameter-group-comparison-20240201/compare_parameter_group1.png)
![compare_parameter_group2](/images/rds-parameter-group-comparison-20240201/compare_parameter_group2.png)

既存のパラメータグループに加えている変更の多くはアップグレード後も適用すべきだと考えられます。結果を確認して設定を検討してください。

### 旧バージョンと新バージョンのデフォルトのパラメータグループを比較

エンジンバージョンが異なると、設定値の変更、設定項目の追加・削除があります。これもパラメータグループの比較を使うことで簡単に調べることができます。
新バージョンのパラメータグループファミリーでデフォルト値のパラメータグループを作成してください。その後同じように旧バージョンのパラメータグループと比較を行うだけです。

![compare_parameter_group3](/images/rds-parameter-group-comparison-20240201/compare_parameter_group3.png)
![compare_parameter_group4](/images/rds-parameter-group-comparison-20240201/compare_parameter_group4.png)

こちらも内容を吟味し、新しいパラメータグループに設定する内容を検討してください。

## DB エンジンバージョンのアップグレード

パラメータグループの検討ができたら DB エンジンバージョンのアップグレードに着手しましょう。一般的にはまず開発環境に適用し、以下の点を確認するとよいでしょう。

- エンジンアップグレードの実施手順
- アップグレード作業時間・サービスダウンタイム
- アップグレード後のアプリケーション動作とパフォーマンス
- 万が一に備えた切り戻し方法・連絡・体制・指揮系統

## まとめ

RDS の運用を任されたが DB の設計や構築はしていないという方に、パラメータグループの存在とパラメータグループの比較によるアップグレード時の調査方法が伝われば幸いです！
