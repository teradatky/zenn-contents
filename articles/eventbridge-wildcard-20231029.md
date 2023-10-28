---
title: "EventBridge のルールにワイルドカードがサポートされました！"
emoji: "🌉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "EventBridge", "CloudTrail", "AmazonSNS"]
published: true
---

## 概要

EventBridge がより便利になるアップデートが行われました！
ルールの中でワイルドカード `*` が使用できるようになり、より柔軟な条件を簡単に表現することが可能になりました。

## 公式ページ

https://aws.amazon.com/jp/about-aws/whats-new/2023/10/amazon-eventbridge-wildcard-filters-rules/
https://aws.amazon.com/jp/blogs/news/aws-weekly-20231002/

## 検証内容

IAM で何かしらのリソースが作成されたら、監査人へ通知する仕組みを作成します。
CloudTrail より `iam:create*` に該当する API コールを EventBridge で拾い、SNS 経由でメール通知します。

![architecture](/images/eventbridge-wildcard-20231029/architecture.png)

:::message
セキュリティ向上を目的とする場合、このような自前の作り込みは避けて SecurityHub や GuardDuty を利用しましょう。
:::

## 構築

:::message alert
IAM の API コールは `us-east-1` リージョンに記録されます。そのため検証リソースはすべてバージニア北部に作成する必要があります。
:::

### SNS

まずは SNS トピックを作成します。タイプはスタンダードを選びます。その他の設定はデフォルトで問題ありません。
次にサブスクリプションの作成をします。プロトコルにEMAILを選び、メールアドレスを設定します。
確認メールが届くので、リンクをクリックします。ステータスが確認済みになればOKです。

![SNS](/images/eventbridge-wildcard-20231029/sns.png)

### EventBridge

次に EventBridge ルールを作成します。イベントバスは `default` でOKです。

![EventBridge](/images/eventbridge-wildcard-20231029/eventbridge1.png)

イベントソースは AWS イベントまたは EventBridge パートナーイベントを選択します。

![EventBridge](/images/eventbridge-wildcard-20231029/eventbridge2.png)

イベントパターンは以下のように設定します。
ここが今回の肝で `"eventName": [{"wildcard": "Create*"}]` と指定しています。

```json
{
  "source": ["aws.iam"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["iam.amazonaws.com"],
    "eventName": [{
      "wildcard": "Create*"
    }]
  }
}
```

![EventBridge](/images/eventbridge-wildcard-20231029/eventbridge3.png)

:::message
イベントタイプの仕様１から直接記入すると、記号が自動エスケープされて意図した記載ができません。「パターンを編集」から直接 `json` を記入します。
:::

:::message alert
単に `"eventName": ["Create*"]` と書いた場合、意図したワイルドカード動作になりません。詳しい記法は以下を参考にしてください。2023/10/29時点では英語ページにのみワイルドカードの記載があります。
@[card](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html#eb-filtering-wildcard-matching)
:::

ターゲットは作成済みの SNS トピックを選択します。

![EventBridge](/images/eventbridge-wildcard-20231029/eventbridge4.png)

以上で構築は完了です！

## 動作確認

適当な IAM ロールを作成します。信頼されたエンティティに EC2 を選択し、ポリシーのアタッチはしません。

![Test](/images/eventbridge-wildcard-20231029/test1.png)

CloudTrail のイベント履歴を確認すると `CreateRole` と `CreateInstanceProfile` が呼び出されていることが分かります。
EC2 を信頼関係に選んだため、IntanceProfile も自動的に作成されます。

![Test](/images/eventbridge-wildcard-20231029/test2.png)

メールを確認すると `Create` から始まる API の呼び出しについて 2 件の通知が届いています。成功です！

![Test](/images/eventbridge-wildcard-20231029/test3.png)

## まとめ

EventBridge のルールでワイルドカードが使えるようになり、より簡単にイベント駆動型システムを組めるようになりそうです。
一方 IAM ポリシーの感覚で `*` をつけるだけでは期待した動作とならないので注意が必要です。
