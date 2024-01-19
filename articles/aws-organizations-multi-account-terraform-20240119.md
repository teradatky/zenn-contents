---
title: "AWS Organizations でマルチアカウント with Terraform"
emoji: "🏢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Terraform"]
published: true
---

## 概要

AWS Organizations で複数アカウントを Terraform で作っていきます。
マルチアカウントにするメリット、および OU 設計のベストプラクティスは以下を参考にしてください。

https://aws.amazon.com/jp/blogs/news/best-practices-for-organizational-units-with-aws-organizations/

## ユースケース

多くの AWS アカウントを運用する必要があるケースに有用です。
払い出し工数の削減や、アカウントの一覧化・可視化を可能にします。

- 新入社員研修で全員にアカウントを払い出し、終了後に削除する
- 事業会社が各プロダクトやサービスごとに開発・検証・本番アカウントを払い出す
- 個人の検証や執筆作業等でまっさらなアカウントを複数利用する

## 構成図

今回はシンプルに dev と prd という 2 つの OU にアカウントをぶら下げる形式で作成します。
またメンバーアカウントが勝手に組織を脱退しないよう、LeaveOrganization を禁じる SCP を設定します。

:::message
SCP は管理アカウントのユーザーやロールには影響を与えません。組織内のメンバーアカウントにのみ影響を与えます。
:::

![organization_architecture](/images/aws-organizations-multi-account-terraform-20240119/organization_architecture.png)

## 構築

Terraform で構築します。以下にコードを掲載します。

:::message alert
AWS アカウントを削除してもメールアドレスの使い回しはできません（永久欠番）。利用するメールアドレスは慎重に選んでください。
:::

:::message
メールアドレスのエイリアスを利用すると、複数のアドレスを簡単に用意できます。Gmail の場合はこちらの「Gmail エイリアスを使用する」を参照してください。

@[card](https://support.google.com/mail/answer/22370?ctx=gsidentifer&sjid=16159587978976700208-AP#)

:::

### リポジトリ全体

```bash:overview
$ tree aws-organizations/
aws-organizations/
├── README.md
├── account.tf
├── organization.tf
├── policy_attachment.tf
├── policy_root_unit.tf
├── setting.tf
└── unit.tf
```

### Organization

```hcl:organization.tf
resource "aws_organizations_organization" "org" {
  aws_service_access_principals = [
    "config.amazonaws.com",
    "cloudtrail.amazonaws.com",
    "ssm.amazonaws.com",
    "health.amazonaws.com",
    "sso.amazonaws.com",
  ]

  enabled_policy_types = [
    "SERVICE_CONTROL_POLICY",
  ]

  feature_set = "ALL"
}
```

### OU

```hcl:unit.tf
resource "aws_organizations_organizational_unit" "prd" {
  name      = "prd"
  parent_id = aws_organizations_organization.org.roots[0].id
}

resource "aws_organizations_organizational_unit" "dev" {
  name      = "dev"
  parent_id = aws_organizations_organization.org.roots[0].id
}
```

### Service Control Policy

```hcl:policy_root_unit.tf
data "aws_iam_policy_document" "root_scp" {
  version = "2012-10-17"
  statement {
    sid = "DenyLeaveOrganization"
    actions = [
      "organizations:LeaveOrganization",
    ]
    effect = "Deny"
    resources = [
      "*"
    ]
  }
}

resource "aws_organizations_policy" "root" {
  name        = "root_policy"
  description = "policy_for_all_accounts"
  content     = data.aws_iam_policy_document.root_scp.json
}
```

```hcl:policy_attachment.tf
resource "aws_organizations_policy_attachment" "root" {
  policy_id = aws_organizations_policy.root.id
  target_id = aws_organizations_organization.org.roots[0].id
}
```

### Account

```hcl:account.tf
# 管理アカウント management-account
# hoge@gmail.com

resource "aws_organizations_account" "prd-account" {
  name              = "prd-account"
  email             = "hoge+prd@gmail.com"
  parent_id         = aws_organizations_organizational_unit.prd.id
  close_on_deletion = true
}

resource "aws_organizations_account" "dev-account" {
  name              = "dev-account"
  email             = "hoge+dev@gmail.com"
  parent_id         = aws_organizations_organizational_unit.dev.id
  close_on_deletion = true
}
```

## 実行

Management Account から terraform を実行します。すると画像のように複数の AWS アカウントが作成できていることがわかります。

:::message
筆者は OIDC を介して管理アカウントへ Terraform Cloud からデプロイしています。
:::

![org_tree](/images/aws-organizations-multi-account-terraform-20240119/org_tree.png)

## アカウント作成後

アカウント作成後は以下手順でログインします。

1. ブラウザで AWS にアクセスし「コンソールにサインイン」をクリック
2. 「ルートユーザー」を選択し、作成したアカウントのメールアドレスを入力し「次へ」をクリック
3. 「パスワードをお忘れですか？」を選択し、画像中の文字を回答し、「Eメールを送信する」をクリック
4. 受領したメールからパスワードをリセット
5. リセットしたパスワードでルートユーザーとしてログインできることを確認

## まとめ

Azure のリソースグループや Google Cloud のプロジェクトと比べると、AWS のマルチアカウントはハードルが高いように思いますが、やってみると意外とすんなり実施可能です。

AWS ではあらかじめアカウントを分割しておかなかったために大変なことになっている案件が散見されます。まずマルチアカウントという戦略と導入の手軽さが伝われば幸いです。

個人が数アカウントを使うだけならコンソールから実施してもよいですが、アカウント数が非常に多かったり、頻繁に払い出しや削除を行う場合はぜひ IaC 化を検討してください。
