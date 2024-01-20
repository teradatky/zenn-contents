---
title: "IAM Identity Center でマルチアカウントでも楽々ログイン！"
emoji: "👟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "IAM"]
published: true
---

## はじめに

AWS IAM Identity Center (旧 AWS Single Sign-On) は 複数の AWS アカウントやアプリケーションへのアクセスを一元的に管理できるサービスです。今回は以下をマネジメントコンソールから作成し、マルチアカウントへのログインを簡略化します。

![sso_architecture](/images/iam-identity-center-multi-account-20240120/sso_architecture.png)

## 前提条件

- AWS Organizations を有効化していること
- 組織にログインしたい AWS アカウントが属していること

## 構築

### 実施作業

以下流れで作業を行っていきます。

1. IAM Identity Center を管理アカウントで有効化
1. IAM Identity Center にユーザーを作成
1. IAM Identity Center にグループを作成
1. グループにユーザーを追加
1. 許可セットを作成
1. AWS アカウントにグループと許可セットを割り当て
1. インスタンス名・アクセスポータルの URL など設定を編集（オプション）

公式の手順はこちら。

https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/useraccess.html

### 設定例

入力項目が少なく、画面の指示に従っていくだけで作成できます。ステップバイステップの解説はせず、簡単な設定例をお見せします。

### ユーザー

ユーザー名・メールアドレス・名・性・表示名を入力してください。追加後は必ずメールから設定を済ませ、ステータスが有効となったか確認してください。

![](/images/iam-identity-center-multi-account-20240120/user.png)

### グループ

グループ名を入力してください。その後グループに作ったユーザーを追加してください。

![](/images/iam-identity-center-multi-account-20240120/group1.png)
![](/images/iam-identity-center-multi-account-20240120/group2.png)


### 許可セット

今回は事前定義された許可セットから AdministratorAccess を選び作成します。

![](/images/iam-identity-center-multi-account-20240120/allow_set1.png)
![](/images/iam-identity-center-multi-account-20240120/allow_set2.png)

### アカウント

アカウントを選択し、割り当てるグループおよび許可セットを選択してください。

![](/images/iam-identity-center-multi-account-20240120/account1.png)
![](/images/iam-identity-center-multi-account-20240120/account2.png)

### 設定

インスタンス名やアクセスポータルの URL を変更することでわかりやすく利用できます。

:::message alert

- インスタンス名は公開されるため、機密情報を含めないでください
- アクセスポータルの URL (サブドメイン) は一度設定すると二度と変えられないため、よく検討してください

:::

![](/images/iam-identity-center-multi-account-20240120/setting.png)

## 動作確認

アクセスポータルにアクセスしてみると、一つのアイデンティティで複数の AWS アカウントにサクッとアクセスできます！すごい！

![](/images/iam-identity-center-multi-account-20240120/sigin-in1.png)
![](/images/iam-identity-center-multi-account-20240120/sigin-in2.png)
![](/images/iam-identity-center-multi-account-20240120/sigin-in3.png)
![](/images/iam-identity-center-multi-account-20240120/sigin-in4.png)
![](/images/iam-identity-center-multi-account-20240120/sigin-in5.png)

別アカウントにアクセスしたい際はもう一度ポータルにアクセスしてください。再認証なくアカウント選択画面に移動します。

## その他

### 改善してほしいところ

今どのアカウントで作業しているのか？が分かりにくいです。画面右上のアカウント表示は `許可セット名/ユーザー名` となるためです。ここをクリックするとアカウント ID が表示されるのですが、マルチアカウント環境で 12 桁の数字とアカウント用途を脳内台帳で突き合わせるのはツライです。

![](/images/iam-identity-center-multi-account-20240120/check.png)

マウスオーバーして待てばアカウントエイリアスが表示されますが ( `@ アカウントエイリアス名` ) そもそも手間なのと表示が小さく見づらいです。デフォルトでアカウントエイリアスが表示されるように期待です。

![](/images/iam-identity-center-multi-account-20240120/account_alias.png)

### フェデレーションの仕組み

AWS アカウントへのログインについて、内部では SAML ベースのフェデレーションが行われています。概要については以下を確認してください。

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_providers_saml.html

これによって非常に嬉しいのが、IAM ユーザーが不要になるということです！各アカウントに余計な IAM ユーザーが作成される、残り続けるといった無法地帯を回避できます。基本的に IAM ユーザーは作らないという運用とすることで、アクセスキーをむやみに発行するオペレーションにも一定の圧をかけることができると思います。

![](/images/iam-identity-center-multi-account-20240120/iam_user.png)

### さらなる利活用

皆さんの勤める企業では、Active Directory などが既に存在するのではないでしょうか？今回は独自にユーザーやグループを AWS 上で作成しましたが、既存のアイデンティティストアを利用すると追加の ID が不要になり、よりよい従業員体験に繋がるでしょう。

![](/images/iam-identity-center-multi-account-20240120/extra1.png)

また SSO は AWS アカウント以外でも利用できます。OAuth 2.0 か SAML 2.0 に対応していればどんなアプリケーションもポータルサイトからアクセスできるようになります。Salesforce のような一般的なビジネスアプリケーションであれば、設定済みのカタログから選択することができます。AWS アカウントだけにとどまらず、様々なサービスに SSO できるよう試してみてはいかがでしょうか？

![](/images/iam-identity-center-multi-account-20240120/extra2.png)
![](/images/iam-identity-center-multi-account-20240120/extra3.png)

## まとめ

アカウントごとにサインイン URL や IAM ユーザー名を控えた Excel を用意してログインする、といった手間が省けて非常に嬉しいソリューションです。みんなでマルチアカウント推進・トイル撲滅していきましょう！
