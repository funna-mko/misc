# Gitメモ

## 1. 設定編

- バージョン確認 `$ git version`
- 内容確認  `$ git config --list --(system|global|local) `
- 設定のレベル
  - system : システム全体の設定 `/etc/gitconfig`
  - global : アカウント共通の設定 `~/.gitconfig`
  - local : 現在居るリポジトリの設定 `./.git/.git/config`

## 未整理

- ローカルリポジトリ作成 `$ git init <Directory> `
  指定したディレクトリの直下に .git フォルダが作られる。
  既に .git フォルダが存在するディレクトリを指定すると初期化する。

- リモートリポジトリ追加

- GitHubから Cloneする時（https編）



- コンフリクト解消 (merge 時に `fatal: refusing to merge unrelated histories`エラーになる時)
`$ git merge --allow-unrelated-histories origin/master` オプションをつける。
エディタでコンフリクトを解消して、マージ⇒コミット





