---
title: "AWS Network Firewall でドメインホワイトリスト形式によるアウトバウンド通信制御"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "NetworkFirewall", "Security", "Proxy"]
published: false
---

## AWS Network Firewall とは

## ユースケース

### 構成図

### 構築

### 動作確認

## 注意点

### セキュリティ対策効果

悪質な内部犯や高度なマルウェア等には効果がない場合があります。これは AWS Network Firewall は Host ヘッダや SNI の server_name を見ているだけで、実際の通信先を検査していないからです。より詳しい解説は以下ページを参照ください。

https://zenn.dev/osprey/articles/networkfirewall-is-not-for-domainbase-restriction

:::message
よくある要件の「アウトバウンド通信制御」を行っていない訳ではないため、Squid 等のプロキシサーバーが必要かはお客様や関係者と相談しましょう。
:::

::::details ドメイン制御を回避してみる

以下コマンドで SNI の内容を偽装します。

```bash
curl https://www.google.com --resolv www.google.com:443:183.79.250.251 -H "Host: yahoo.co.jp" --insecure -v
```

:::message
`183.79.250.251` は Yahoo! JAPAN の IP アドレス（のひとつ）です。
:::

![spoofing](/images/network-firewall-domain-whitelist-20231105/spoofing.png)
*Google への通信を偽装することで、Yahoo に通信できてしまう*
::::

### 価格

意外と高額になりがちです。固定費は工夫して削減することが難しく、小～中規模なプロジェクトでは支出の多くを占める可能性があります。以下は東京リージョンでの利用例です。

| Type                    | Amount    | Pricing      | Total     | Remarks            |
| ----------------------- | --------- | ------------ | --------- | ------------------ |
| Endpoint Charges        | 720 h * 2 | 0.395 USD/h  | 568.8 USD | 30 days, Multi-AZ  |
| Data Processing Charges | 3000 GB   | 0.065 USD/GB | 195.0 USD | 30 days, 100GB/day |

:::message
単に AWS リソースの稼働コストだけであれば、ALB + EC2 (Squid) の方が安いです。検討時には EC2 や Squid の運用コストを考慮しましょう。
:::

## まとめ

注意点のように気をつけるべきポイントはありますが、ドメインによるホワイトリスト通信制御として利用可能です。選択肢の一つとして持っておくとよいでしょう。
