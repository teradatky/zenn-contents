---
title: "EventBridge のフィルターにプレフィックス・サフィックス・以外・大文字小文字無視の条件が追加されました！"
emoji: "🌉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "EventBridge"]
published: false
---

## 概要

Twitter で EventBriidge のルールのフィルタリングがパワーアップしたとの情報をキャッチし、実際に調べてみました！

https://twitter.com/nickste/status/1753513030984536380

## ドキュメント確認

AWS の公式ドキュメントを確認すると、Filter types に `Prefix matching`, `Suffix matching`, `Anything-but matching`, `Equals-ignore-case matching` が追加されています！

https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html

一方 2024/02/04 時点では、まだ AWS Blog や 週刊 AWS には紹介がありませんでした。

## 実際に確認

EventBridge で実際にイベントパターンを作って確認します。今回はサンプルイベントに `EC2 Instance Launch Unsccessful` イベントを選び、ASG 名を `anything-but` 条件でフィルタリングしてみます。

シチュエーションとしては基本的にはイベントを発火したいが、障害を模したリソース（プレフィックスが broken）では発火させないというケースを想定しています。

![](/images/eventbridge-new-filter-20240204/sample-event.png)
![](/images/eventbridge-new-filter-20240204/filtering-result1.png)
![](/images/eventbridge-new-filter-20240204/filtering-result2.png)

```json:event-pattern.json
{
  "source": ["aws.autoscaling"],
  "detail-type": ["EC2 Instance Launch Unsuccessful"],
  "detail": {
    "AutoScalingGroupName": [{
      "anything-but": {
        "prefix": "broken"
      }
    }]
  }
}
```

ASG 名のプレフィックスが broken のイベントのみ一致しないため、新しいフィルター条件 `anything-but` が正しく機能していることが確認できました！

## まとめ

EventBridge 単体でフィルタリングできていなかった処理をシンプルにできる可能性があり、嬉しいアップデートだと思います！

よろしければ EventBridge のフィルタリングにワイルドカードを使った例もどうぞ。

https://zenn.dev/teradatky/articles/eventbridge-wildcard-20231029
