---
title: "Terraform のディレクトリ構成に関する戦略と所感"
emoji: "📂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Terraform"]
published: false
---

## Terraform のディレクトリ構成と戦略

### ベタ書き（ディレクトリ分割）

```text:tree.txt
takuya@DESKTOP:~/terraform$ tree .
.
└── env
    ├── dev
    │   ├── application.tf
    │   ├── database.tf
    │   ├── locals.tf
    │   └── provider.tf
    ├── prd
    │   ├── application.tf
    │   ├── database.tf
    │   ├── locals.tf
    │   └── provider.tf
    └── stg
```

#### メリット

- 環境差分に強く考えることが少ない
- 直感的でスキルが高くないメンバでも読み書きできる

#### デメリット

- 同じコードを繰り返し書く割合が高い
- 変更時の抜け漏れが起きやすい

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
        ├── outputs.tf
        └── variables.tf
```

#### メリット

- 共通箇所の繰り返しを避けることができる
- ルートモジュール（Module 呼び出し側）の見通しがよい

#### デメリット

- Module に切り出す単位や粒度の設計が難しい
- 環境差分や汎化を意識すると、ベタ書きより冗長になる

### Workpace 分割

```text:tree.txt
takuya@DESKTOP:~/terraform$ tree .
.
├── application
│   └── main.tf
├── database
│   └── main.tf
└── vars
    ├── dev.tf
    ├── prd.tf
    └── stg.tf
```

#### メリット

- 設定変更時の環境変更漏れが起こりにくい
- 環境ごとの変数が切り出され見通しがよい

#### デメリット

- 環境差分を受け入れようとすると複雑になる（`count` 地獄）
- Workspace ごとの変数使い分けや、操作対象の意識が必要

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

インフラチームが開発体制や方針に介入できる、高スキルメンバーが揃えられるといった場合でない限り、ベタ書きが最も柔軟に対応できるため安定しやすいという印象です。

## 参考文献

https://future-architect.github.io/articles/20190903/
https://qiita.com/reireias/items/253529c889cafb3fa4c7
