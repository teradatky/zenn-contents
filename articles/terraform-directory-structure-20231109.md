---
title: "Terraform のディレクトリ構成戦略"
emoji: "📂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terraform", "AWS"]
published: true
---

## Terraform のディレクトリ構成と戦略

Terraform のディレクトリ構成は主に 3 パターンあります。それぞれの戦略があり、最も優れたものは一概に決められません。利用するプロジェクトの性質に合わせて選択することが大事です。

### ベタ書き（ディレクトリ分割）

```text:tree.txt
takuya@DESKTOP:~/terraform$ tree .
.
└── env
    ├── dev
    │   ├── application.tf
    │   ├── database.tf
    │   ├── loadbalancer.tf
    │   └── provider.tf
    ├── prd
    │   ├── application.tf
    │   ├── database.tf
    │   ├── loadbalancer.tf
    │   └── provider.tf
    └── stg
```

```hcl:env/prd/application.tf
resource "aws_instance" "main" {
  ...
  instance_type = "m5.large"
  tags = {
    Name = "prd-commerce-ec2"
  }
}
...
```

```hcl:env/dev/application.tf
resource "aws_instance" "main" {
  ...
  instance_type = "t3.small"
  tags = {
    Name = "dev-commerce-ec2"
  }
}
...
```

:::message
何度も出てくる値は `locals` でまとめると良いでしょう。

```hcl:env/prd/locals.tf
locals {
  name = "commerce" # local.name で参照
  env  = "prd"      # local.env で参照
  ...
}
```

:::

#### メリット

- 環境差分に強く考えることが少ない
- 直感的でスキルが高くないメンバでも読み書きできる

#### デメリット

- 同じコードを繰り返し書く割合が高い
- 変更時の抜け漏れが起きやすい

#### コメント

同じコードが何度も現れ、DRY の原則に従っていない点が目につきます。しかし無理な共通化によって拡張が困難になることもありません。開発環境だけ X システムと Y システムは共用で、ステージング環境 C 面のみ負荷試験を行う Z 社と接続して…といった環境差分に困ることもありません。愚直ですが柔軟な構成です。

### Module 分割

```text:tree.txt
takuya@DESKTOP:~/terraform$ tree .
.
├── env
│   ├── dev
│   │   ├── main.tf
│   │   ├── locals.tf
│   │   └── provider.tf
│   ├── prd
│   │   ├── main.tf
│   │   ├── locals.tf
│   │   └── provider.tf
│   └── stg
└── modules
    └── web_service
        ├── application.tf
        ├── database.tf
        ├── loadbalancer.tf
        ├── outputs.tf
        └── variables.tf
```

```hcl:modules/web_service/application.tf
resource "aws_instance" "main" {
  ...
  instance_type = var.instance_type
  tags = {
    Name = join("-", [var.env, var.name, "ec2"])
  }
}
...
```

```hcl:env/prd/main.tf
module "commerce" {
  source        = "../../modules/web_service"
  env           = "prd"
  name          = "commerce"
  instance_type = "m5.large"
  ...
}
```

```hcl:env/dev/main.tf
module "commerce" {
  source        = "../../modules/web_service"
  env           = "dev"
  name          = "commerce"
  instance_type = "t3.small"
  ...
}
```

#### メリット

- 共通箇所の繰り返しを避けることができる
- ルートモジュール（Module 呼び出し側）の見通しがよい

#### デメリット

- Module に切り出す単位や粒度の設計が難しい
- Module の引数やロジックを柔軟にすると、ベタ書きより冗長
- Module の引数やロジックを制約すると、環境差分についていけない

#### コメント

共通化できるところはモジュール化し、個別の環境差分も無理なく取り入れられる形です。個人的には最も綺麗で好ましいと感じます。一方モジュールの切り出し方や粒度が難しく、構成変更や機能追加に伴い改修工数がかさんだり `state mv` が必要になることがしょっちゅうあります。

### Workpace 分割

```text:tree.txt
takuya@DESKTOP:~/terraform$ tree .
.
├── web_service
│   ├── application.tf
│   ├── database.tf
│   ├── loadbalancer.tf
│   └── provider.tf
└── vars
    └── web_service.tf
```

```hcl:web_service/application.tf
resource "aws_instance" "main" {
  ...
  instance_type = local.instance_type[terraform.workspace]
  tags = {
    Name = join("-", [terraform.workspace, "commerce", "ec2"])
  }
}
...
```

```hcl:vars/web_service.tf
locals {
  instance_type = {
    prd = "m5.large"
    dev = "t3.small"
    ...
  }
  ...
}
```

:::message
Terraform Cloud における Workspace は別の意味を持つ語なので注意してください。
@[card](https://developer.hashicorp.com/terraform/cloud-docs/workspaces#terraform-cloud-vs-terraform-cli-workspaces)
:::

#### メリット

- 設定変更時の環境変更漏れが起こりにくい
- 環境ごとの変数が切り出され見通しがよい

#### デメリット

- 環境差分を受け入れようとすると複雑になる（`count` 地獄）
- Workspace ごとの変数使い分けや、操作対象の意識が必要

:::details count 地獄の例

```hcl
...
# 開発環境の CI サーバーは XX サーバーと同居するので不要
resource "aws_instance" "ci_server" {
  count = terraform.workspace == "dev1" || "dev2" || "dev3" ? 0 : 1
  ...
}

# ステージング C 面のみ Z 社と接続（負荷試験用）
resource "aws_vpc_peering_connection" "z_corp" {
  count = terraform.workspace == "stg3" ? 1 : 0
  ...
}

# ステージング C 面のみ Z 社と接続（負荷試験用）
resource "aws_route" "z_corp" {
  count = terraform.workspace == "stg3" ? 1 : 0
}
...
```

:::

#### コメント

一意のコードにより環境が作成されるため、環境差分や変更の抜け漏れを発生させません。常に同一構成でテストされたものが本番環境にデプロイされるという安心感があります。一方環境差分を認める場合、`count` による条件分岐が発生し、管理しにくくなります。この戦略を理由に「環境差分を作らせない」という進め方ができるなら心強い反面、環境差分に対応するとなった場合の負荷はかなり高いと言わざるを得ません。

## まとめ

### おすすめ戦略

| 戦略                         | いつ使う                       | おすすめ度 |
| ---------------------------- | ------------------------------ | ---------- |
| ベタ書き（ディレクトリ分割） | どの戦略を使うか迷うとき       | ★★★     |
| Module 分割                  | 共通部分を綺麗に設計できるとき | ★★       |
| Workspace 分割               | 環境差分を許容しないとき       | ★         |

### 所感

自社開発でなくSIerとして協力する立場では、以下実態になるプロジェクトが多いです。こうなると Module 分割や Workspace 分割のよさが引き出せません。

- 多ベンダー・多チームの体制により統制が効きにくい
- 他チーム・他ベンダー・他システム・他アカウントと合流が起こる
- インフラの合理化よりアプリや業務チームの要望が優先
- 協力会社のメンバーが多くスキルがバラバラ
- 複数の環境面があって共通の要素がほとんどない
- 環境面はコピーではなく複数チームが別々の作業をする場という考え

インフラチームに高スキルメンバーが揃えられ、開発体制や方針に積極的に介入できるといった場合でない限り、ベタ書きが最も柔軟に対応できるため安定しやすいという印象です。

## 参考文献

https://future-architect.github.io/articles/20190903/
https://qiita.com/reireias/items/253529c889cafb3fa4c7
