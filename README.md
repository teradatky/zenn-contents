# Zenn CLI

- [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

## 始め方

```bash
# レポジトリの clone
git clone https://github.com/teradatky/zenn-contents.git

# Zenn CLI のインストール
cd zenn-contents
npm install

# 記事作成
# slug は全ユーザーの記事で一意である必要があるので日付をつけると良い
npx zenn new:article --slug my-article-20231026

# プレビュー
npx zenn preview
```

## 参考

- [GitHubリポジトリでZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)
- [Zenn CLIで記事・本を管理する方法](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [ZennのMarkdown記法一覧](https://zenn.dev/zenn/articles/markdown-guide)
