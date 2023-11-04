---
title: "AWS Network Firewall でドメインによるホワイトリスト通信制御"
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

```bash
curl https://www.google.com --resolv www.google.com:443:183.79.250.251 -H "Host: yahoo.co.jp" --insecure -v
```

:::message
`183.79.250.251` は Yahoo! JAPAN の IP アドレス（のひとつ）です。
:::

![spoofing](/images/network-firewall-domain-whitelist-20231105/spoofing.png)
*Google への通信を偽装することで、Yahoo に通信できてしまう*

より詳しい解説が以下ページに載っています。

https://zenn.dev/osprey/articles/networkfirewall-is-not-for-domainbase-restriction

### 価格
