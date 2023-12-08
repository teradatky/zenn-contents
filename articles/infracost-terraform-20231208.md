---
title: "Infracost で Terraform コードからコストを試算してみる"
emoji: "💵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Terraform", "infracost"]
published: false
---

## Infracost

Infracost は IaC コードからクラウドサービスの利用料を試算するツール/サービスです。執筆時点では Terraform のみの対応ですが、ロードマップでは Pulumi, CDK, Bicep に対応予定のようです。

https://www.infracost.io/
https://github.com/infracost/infracost

## 導入

VSCode および公式サイトでの利用法を記載します。

:::message
CLI 利用や CI/CD ワークフローへの導入も可能です。公式サイトを確認ください。
:::

VSCode の拡張機能から `Infracost` を検索してインストールします。その後画面の指示に従いサインアップします。私は GitHub アカウントを利用しました。後述の Infracost Cloud でのコスト試算を行う場合は、連携する VCS を選択するまで実施してください。

:::details セットアップ画面
![](/images/infracost-terraform-20231208/vscode_install.png =400x)
![](/images/infracost-terraform-20231208/login.png =300x)
![](/images/infracost-terraform-20231208/integrations.png)
:::

## VSCode でコスト試算

Terraform コードを含むフォルダを VSCode で開くと自動でコスト試算を表示してくれます。

![](/images/infracost-terraform-20231208/vscode1.png)
![](/images/infracost-terraform-20231208/vscode2.png)


:::message
VSCode のワークスペース機能で複数のフォルダを開いていると、拡張機能が正しく動作しません。これはかなり残念ポイントです。

@[card](https://github.com/infracost/vscode-infracost/issues/1)

![](/images/infracost-terraform-20231208/bug1.png)
![](/images/infracost-terraform-20231208/bug2.png)
:::

## Infracost Cloud でコスト試算

Infracost Cloud ではより詳しく確認できます。サインアップしてから 2 週間無料で利用できます。期限を過ぎると CLI を除く機能は有料のサブスクリプションが必要になるようです。

https://www.infracost.io/pricing/

ダッシュボードでは 5 項目について、コスト最適化設定の達成率を表示してくれます。

![](/images/infracost-terraform-20231208/dashboard.png)

以下リポジトリ個別を調査してみます。もちろんコスト試算はしてくれるのですが、なにやら FinOps policies と Tagging policies に違反するリソースがあることが分かりました。

https://github.com/teradatky/aws-network-firewall-sample

![](/images/infracost-terraform-20231208/repo_overview.png)
![](/images/infracost-terraform-20231208/finops_policy.png)
![](/images/infracost-terraform-20231208/tagging_policy.png)
![](/images/infracost-terraform-20231208/project.png)

Infracost が用意するポリシーが気になるので、調べてみましょう。画面上部の Governance から FinOps policiesをクリックします。指摘済みのCloudWatch Logs のログ保持期間以外にも様々なポリシーがあることが分かります。個別にポリシーの ON/OFF もできるようです。

![](/images/infracost-terraform-20231208/finops_policies1.png)
![](/images/infracost-terraform-20231208/finops_policies2.png)
![](/images/infracost-terraform-20231208/finops_policies3.png)
![](/images/infracost-terraform-20231208/finops_policies4.png)
![](/images/infracost-terraform-20231208/finops_policies5.png)

Tagging policies も確認します。デフォルト設定は以下の通りです。タグ付けを強制はクラウドサービス側で行うのが定石かと思いますが、Infracost 側で行うこともできそうです。

![](/images/infracost-terraform-20231208/tagging_policies1.png)
![](/images/infracost-terraform-20231208/tagging_policies2.png)

## まとめ

Infracost を利用すると、IaC化さえしていればコスト試算が簡単に行えることが分かりました。一方 VSCode の拡張機能はワークスペース利用時に正常動作しなかったり、一部計算の表示がおかしかったりすることが分かりました。改善に期待です。

Infracost Cloud はコスト試算に加え、コスト最適化のベストプラクティスを指摘してくれるのため、一度目を通す価値はあると思います。ただしあくまでコスト影響のあるパラメータに対する指摘であり、ワークロードに対する最適なアーキテクチャを教えてくれるわけではありません。これは業務の中で身につけていきたいです。
