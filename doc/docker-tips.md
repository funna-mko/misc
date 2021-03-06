# docker に関する覚書
## 1. インストール
### 1.1 Linux
### 1.2 Windows
    Windows10 Professional エディションでのみ実行可能
    （Home エディションで実行する場合は特別なやり方があるらしい）

    1) 仮想マシンを有効化 （有効化後Windows再起動が必要）
    2) Docker for windows インストール

## 2. 概念
    1) イメージ     特定の実行環境を保存した物。基本的なイメージは DockerHubから取得可能
    2) コンテナ     イメージから起動した仮想マシン（実行環境）
    3) ストレージ   コンテナに割り当てるストレージ

   - docker commit でコンテナをイメージとして保存できる。
   - 保存して作成したイメージを自分の docker hub にアップロード可能

|コマンド|意味|
|:----|:------------------------------------|
|#docker pull [imageパス] |DockerHubからイメージを取得|
|#docker images |取得済イメージ一覧表示|
|#docker rmi [イメージID] |イメージ削除|
|#docker run -d -it --name [コンテナ名] [イメージ名] |イメージ起動 *起動引数の説明は後述|
|#docker ps -a |コンテナ一覧 [-a 起動中以外も含む] |
|#docker exec -it [コンテナ名] bash |指定コンテナのターミナル起動|
|#docker stop [コンテナ名] |指定コンテナ停止|
|#docker start [コンテナ名] |指定コンテナ再開|
|#docker rm [コンテナID] |指定コンテナ削除|

* コンテナ起動時オプションの指定方法

-v [ホスト側フォルダ名]:[コンテナ内フォルダ名] ... コンテナへのフォルダ割り当て指定
-p [ホスト側ポート番号]:[コンテナ側ポート番号] ... ポートのりダイレクト指定




## 参考URL
 - Docker入門     https://knowledge.sakura.ad.jp/13265/


