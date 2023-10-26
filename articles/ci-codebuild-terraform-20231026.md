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

CodeBuild のビルドプロジェクトは 2 つ作成します。
それぞれプルリクエスト作成/更新とマージに対応します。

![構成図](/images/ci-codebuild-terraform-20231026/architecture.png)

:::message
`terraform apply` が同時に実行される可能性がある場合（CI 以外にローカルから apply を許可する場合など）は tfstate の破損を防ぐために　DynamoDB による state lock を追加してください。
:::

:::details GitHub Actionsについて
GitHub Actions を利用すると CICD に関連するリソースが GitHub に閉じるため、よりシンプルに利用できます。筆者の参画する案件では GitHub Enterprise（いわゆるオンプレ版）が提供されていますが、GitHub Actions を利用するためにセルフホストランナーを用意する必要がありました。インフラを自前で用意することを嫌ったため、チームの AWS アカウントで CodeBuild を利用することとしました。基本的な概念は同じため、参考にしてください。
:::

### tfnotify

CICD ワークフローに Terraform の plan/apply 結果を通知してくれるツールです。

https://github.com/mercari/tfnotify

## 事前準備

### リポジトリ

GitHub にて新規にリポジトリを作成します。
プライベートリポジトリで問題ありません。

![アクセストークン](/images/ci-codebuild-terraform-20231026/repo_create.png)

### アクセストークン

GitHub にて 個人のアクセストークンを払い出します。
tfnotify が GitHub に通知をするために利用します。
`Settings > Developer Settings > Personal access tokens > Tokens (classic)` より作成できます。
権限は `repo:status` `public_repo` のみで大丈夫です。

![アクセストークン](/images/ci-codebuild-terraform-20231026/ghp_token.png)

:::message
ウィンドウを閉じるとトークンは二度と確認できません。きちんとメモしておきましょう。
:::

### IAM

Terraform 実行に必要な権限を付与した IAM ロールを作成します。
本記事では IAM ポリシー `AmazonS3FullAccess` を利用します。
信頼関係では CodeBuild を許可しましょう。

![IAMロール](/images/ci-codebuild-terraform-20231026/iam_role1.png)
![IAMロール](/images/ci-codebuild-terraform-20231026/iam_role2.png)
![IAMロール](/images/ci-codebuild-terraform-20231026/iam_role3.png)

### S3

Terraform のバックエンド用 S3 バケットを作成します。
デフォルト値から変える設定は特にありません。

![S3バケット](/images/ci-codebuild-terraform-20231026/s3_tfstate.png)

## 構築

サンプルコードはこちら。

https://github.com/teradatky/ci-codebuild-terraform-20231026

### GitHub

リポジトリを参考に、CI 用のコードをプッシュします。
Terraform コードは `main.tf` を除いてください。

CodeBuild が利用する `buildspec_plan.yml` は以下です。
各フェーズでツールのインストール、 `terraform init` 、 `terraform plan | tfnotify` をしています。
apply を行う buildspec はほぼ同じ形式のため、リポジトリを確認してください。

```yml
version: 0.2

env:
  variables:
    TFDIR: "terraform"
    TFNCONF: "codebuild/tfnotify.yml"
    TITLE: "Terraform Plan"
    MSG: "Plan detail via tfnotify"

phases:
  install:
    commands:
      # terraform
      - curl -sL https://releases.hashicorp.com/terraform/1.6.2/terraform_1.6.2_linux_amd64.zip > terraform.zip
      - unzip terraform.zip
      - cp terraform /usr/local/bin
      # tfnotify
      - curl -sL https://github.com/mercari/tfnotify/releases/download/v0.8.0/tfnotify_linux_amd64.tar.gz > tfnotify.tar.gz
      - tar -zxvf tfnotify.tar.gz
      - cp tfnotify /usr/local/bin
  pre_build:
    commands:
      - terraform -chdir="${TFDIR}" init -no-color
  build:
    commands:
      - PLAN=$(terraform -chdir="${TFDIR}" plan -no-color 2>&1)
      - echo "${PLAN}" | tfnotify --config "${TFNCONF}" plan --title "${TITLE}" --message "${MSG}"
```

:::message
CI が頻繁に実行される場合は `terraform` `tfnotify` のインストールを毎回行わずに済むよう、カスタムイメージを利用しましょう。`docker build` を行い ECR にプッシュすることで CodeBuild から利用可能です。
:::

tfnotify 用の設定ファイルは以下です。
こちらは plan と apply が 1 ファイルにまとめられます。

```yml
---
ci: codebuild
notifier:
  github:
    token: $GITHUB_TOKEN
    repository:
      owner: "teradatky"
      name: "ci-codebuild-terraform-20231026"
terraform:
  plan:
    template: |
      {{ .Title }} <sup>[CI link]( {{ .Link }} )</sup>
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code>{{ .Body }}
      </pre></code></details>
  apply:
    template: |
      {{ .Title }}
      {{ .Message }}
      {{if .Result}}
      <pre><code>{{ .Result }}
      </pre></code>
      {{end}}
      <details><summary>Details (Click me)</summary>

      <pre><code>{{ .Body }}
      </pre></code></details>
```

### CodeBuild

ビルドプロジェクトを 2 つ作成します。
「プライマリソースのウェブフックイベント」が変わることに注意します。

- plan 用ビルドプロジェクト
  - `PULL_REQUEST_CREATED` `PULL_REQUEST_UPDATED` `PULL_REQUEST_REOPEND`
- apply 用ビルドプロジェクト
  - `PULL_REQUEST_MERGED`

以下に plan 用ビルドプロジェクトを作成するスクリーンショットを載せています。
apply 用のリソースのスクリーンショットは省略しますが、同様に作成します。

:::message
デフォルト値から変更していない箇所は、スクリーンショットを撮っていません。
名前やログ出力先などもビルドプロジェクトに合わせて適宜変更してください。
:::

:::message
個人用の AWS アカウントでない場合、アクセストークンはプレーンテキストではなく Secrets Manager を利用してください。
:::

![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild1.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild2.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild3.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild4.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild5.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild6.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild7.png)
![CodeBuild](/images/ci-codebuild-terraform-20231026/codebuild8.png)

## CICD ワークフロー例

実際に Terraform コードをプッシュし、プルリクエストを作成/マージしてみます。

