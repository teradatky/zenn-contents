---
title: "AWS CodeBuild と GitHub で実現する Terraform CICD 入門"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CICD", "AWS", "CodeBuild", "Terraform", "GitHub"]
published: false
---

AWS CodeBuild と GitHub による Terraform の CICD 実装例をご紹介します。

## 想定読者

- Terraform の CICD ワークフローを実装したい
- AWS CodeBuild の具体的な使用例をみたい

## 構成図

![構成図](/images/ci-codebuild-terraform-20231026/architecture.png)

:::details GitHub Actionsについて
GitHub Actions を利用すると CICD に関連するリソースが GitHub に閉じるため、よりシンプルに利用できます。筆者の参画する案件では GitHub Enterprise（いわゆるオンプレ版）が提供されていますが、GitHub Actions を利用するためにセルフホストランナーを用意する必要がありました。インフラを自前で用意することを嫌ったため、チームの AWS アカウントで CodeBuild を利用することとしました。基本的な概念は同じため、参考にしてください。
:::

### tfnotify

CICD ワークフローに Terraform の plan/apply 結果を通知してくれるツールです。

https://github.com/mercari/tfnotify


## 事前準備

### リポジトリ

GitHub にて新規にリポジトリを作成してください。
プライベートリポジトリで問題ありません。

### アクセストークン

GitHub にて 個人のアクセストークンを払い出してください。
tfnotify が GitHub に通知をするために利用します。
`Settings > Developer Settings > Personal access tokens > Tokens (classic)` より作成できます。

### IAM

Terraform 実行に必要な権限を付与した IAM ロールを作成してください。
本記事では IAM ポリシー `AmazonS3FullAccess` を利用します。
信頼関係では CodeBuild を許可してください。

## 構築

### GitHub

### CodeBuild

## CICD ワークフロー例
