---
title: "閏年（うるうどし）の閏日（うるうび）にログ転送のエラーが大量発生した話"
emoji: "📅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "CloudWatch", "閏年"]
published: false
---

## 概要

閏年の閏日に CloudWatch Logs エージェント `awslogs` （統合 CloudWatch エージェント `amazon-cloudwatch-agent` ではない）によるログ転送が大量のエラーを吐いてしまった件について記載。

## 事象

2024/02/29 になった瞬間より、以下のような大量のログエラーを検出。

```text
2024-02-29 03:30:03,694 - cwlogs.push.reader - WARNING - 2489 - Thread-575 - Fall back to previous event time:
{'timestamp': 1709132101000, 'start_position': 26849L, 'end_position': 26920L}, previousEventTime: 1709132101000,
reason: timestamp could not be parsed from message.
```

```text
2024-02-29 03:40:09,779 - cwlogs.push.reader - WARNING - 2489 - Thread-578 - Fall back to current time:
{'timestamp': 1709145609779, 'start_position': 499L, 'end_position': 569L},
reason: timestamp could not be parsed from message.
```

## 原因

ログフォーマットと CloudWatch Logs エージェント `awslogs` の組み合わせの問題だった。

- `/var/log/messages`, `/var/log/secure` などで年を含まない `datetime_format` を使用
- `awslogs` では、年を含まない `datetime_format` に対する補完処理をしている
  - Python の `datetime.datetime.strptime` でログの日時をパース
  - 年が含まれない場合、1900 年としてパースされる
  - `datetime_format` に年が含まれていなければ、1900 を今日の年で置き換える
- 閏年の閏日 (2/29) 以外では問題なく動作していた
- 閏年の閏日 (2/29) のみ日時のパースに失敗する
  - 1900 年は閏年ではなく 1900/2/29 は不正な日付のため

### datetime.strptime について

> For the datetime.strptime() class method, the default value is 1900-01-01T00:00:00.000: any components not specified in the format string will be pulled from the default value.

> Passing datetime.strptime('Feb 29', '%b %d') will fail since 1900 is not a leap year.

https://docs.python.org/3/library/datetime.html#datetime.datetime.strptime

## 経過

Warning が大量に発生しているだけで、ログの転送とタイムスタンプには影響はなかった（現在時刻にフォールバックされるため）。また閏日を過ぎるとログエラーは発生しなくなった。

## ポストモーテム

この事象を発生させてしまった起因。

- `awslogs` を使い続けたままで `amazon-cloudwatch-agent` （推奨）に移行していないこと
- 2020/2/29 にも同事象が起きていたが、Issue に起票したきり放置されていたこと

上記を受けた教訓。

- 塩漬けにもリスクは伴うことを認識すべき
- 課題管理などの基本所作が大事

## まとめ

基盤でも構成変更やアクセス負荷以外が起因でトラブルが起きることがあるため、その心づもりとトラブルシュート経験を大切に！
