---
title: "AWS Systems Manager ドキュメント (SSM ドキュメント) で EC2 をプロビジョニング "
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "SystemsManager", "EC2", "Ansible"]
published: false
---

## まえがき

本記事では SSM ドキュメントを利用して、EC2 をプロビジョニングする例を紹介します。ここではホスト名の設定を行います。

## 構成図

![architecture](/images/ssmdocument-ec2-provisioning-20240107/architecture.png)

:::message
Ansible を実行するだけであれば、AWS が用意している `AWS-ApplyAnsiblePlaybooks` を利用するのが簡単です。入出力をカスタマイズしたり、自作の SSM オートメーションと組み合わせることを想定しています。
:::

### SSM ドキュメント採用理由

- プロビジョニング用のサーバー (Ansible, Jenkins, etc...) が不要になる
- プロビジョニング実行やログ確認にサーバーへのログインが不要になる

### Ansible 採用理由

- 手作業による誤りや漏れを防止できる
- シェルスクリプトと比べ、冪等性を担保しやすい

### GitHub 採用理由

- Playbook を各サーバーに展開しやすい (`git clone` するだけ)
- 変更内容の確認やレビューがしやすい

## SSM ドキュメント作成

以下リポジトリを参考にしてください。Terraform を利用しています。

https://github.com/teradatky/ssm_document_sample

https://github.com/teradatky/ssm_document_sample/blob/main/terraform/modules/ssm_document/ssm_document_apply_ansible.tf

https://github.com/teradatky/ssm_document_sample/blob/main/terraform/modules/ssm_document/template_apply_ansible.yml

## SSM ドキュメント実行

まずプロビジョニング前の EC2 を確認してみます。セッションマネージャーで接続しホスト名を調べると、自動で設定されるものになっています。

![connect ec2](/images/ssmdocument-ec2-provisioning-20240107/connect_ec2.png)
![hostname before](/images/ssmdocument-ec2-provisioning-20240107/hostname_before.png)

では SSM ドキュメントを実行してみましょう。マネジメントコンソールから作成したドキュメントを選択します。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc1.png)
![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc2.png)

ホスト名の入力とターゲットのインスタンスの選択をします。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc3.png)

ログを S3 ではなく CloudWatch Logs に書き込むよう選択します。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc4.png)

残りはそのままで実行をクリックします。

![exec ssmdoc](/images/ssmdocument-ec2-provisioning-20240107/exec_ssmdoc5.png)

正常に実行できたか確認しましょう。

![ssmdoc result](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_result1.png)
![ssmdoc result](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_result2.png)

ログが少々見にくいので、CloudWatch Logs からも確認してみましょう。

![ssmdoc log](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_log1.png)
![ssmdoc log](/images/ssmdocument-ec2-provisioning-20240107/ssmdoc_log2.png)

実際にホスト名が変更できているか確認します。想定通りの結果が得られました！

![hostname after](/images/ssmdocument-ec2-provisioning-20240107/hostname_after.png)

## 実案件では

私の参画案件では、EC2 作成も含めて SSM オートメーションによる自動化を行っています。現在の AWS リソースや許可された値から入力値をリスト形式で選択できたり、複雑な入力を Python でパースするフェーズを備えています。その後、入力値に従い EC2 が起動します。

EC2 起動後は今回同様 Ansible を実行する SSM ドキュメントが呼び出され、入力値およびコンプライアンスに準じる設定がなされます。実際には以下のような設定を行っています。

- パッケージインストール
- スクリプト配置
- audit 設定
- cloud-init 設定
- sshd 設定
- logrotate 設定
- sudoers 設定
- LDAP 認証設定
- タイムゾーン設定
- ホスト名設定
- MTU 設定
- ログファイル パーミッション設定
- 作業ログ（実行コマンド）収集設定

この SSM ドキュメントは Terraform によって 100 を超える AWS アカウントに展開されており、コンプライアンス準拠に役立っています。

## まとめ

SSM ドキュメントを用いることで、サーバーレスで EC2 を設定することが可能です。実案件では以前は Jenkins ジョブを利用していたので、このためだけにサーバーを管理する必要がなくなりました！（諸事情で Jenkis は撤去できていませんが…）プロビジョニング用にサーバーを増やす前に、一度 Systems Manager の利用を検討してみてください。
