# Jetson Tips

## 1.環境構築編
### 1.1 カーネルアップグレード
 ｰ カーネルバージョン確認 ```$ uname -a```
 - L4T(Jetson Linux Driver Package)バージョン確認 ```$ cat /etc/nv_tegra_release```

 ### 捕捉1. apt 使い方
 ``` $sudo apt update``` リポジトリ一覧を更新
 ``` $sudo apt upgrade``` アップグレード可能なパッケージを更新
 ``` $sudo apt search {パッケージ名}``` パッケージ検索
 ``` $sudo apt list --installed``` インストール済みパッケージ一覧

## 2.イメージバックアップ
- Jetson から microSD を抜いて Linux PC のカードスロットに差し込む。
  以下、Linux PC 上の作業手順
- マウントポイントの確認
    ```
    [root@localhost jetson-nano]# lsblk
    NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda            8:0    0 238.5G  0 disk 
    (snip)
    sdb            8:16   0 465.8G  0 disk 
    (snip)
    loop0          7:0    0  90.8M  1 loop 
    mmcblk0      179:0    0 116.8G  0 disk    # ←ここ
    ├─mmcblk0p1  179:1    0 116.8G  0 part /run/media/mikio/    cf9d96ca-f7c4-45f5-9064-65234
    ├─mmcblk0p2  179:2    0   128K  0 part 
    (snip)
    └─mmcblk0p14 259:6    0   128K  0 part 
    ```
- dd コマンドでバックアップ （sudo で実行すること）
    ```
    [root@localhost jetson-nano]# dd if=/dev/mmcblk0 bs=64k | gzip -c > ./funa-jsn_img_20200104.gz
    ```
- リストア (実際は未だやってない)
    ```
    # gunzip -c /tmp/backup/funa-jsn_img_20200104.gz | dd of=/dev/mmcblk0 bs=64k
    ```

## 3.ssh接続
### 3.1 パスワードを使った接続
- Host側のIPアドレスを調べる。
  ``` $ ip a``` とかを使って
- クライアントから ```$ ssh funa@192.168.0.11``` とかで接続
  取り敢えず、ローカルのユーザー名／パスワードで接続可能
 ```
 [mikio@localhost ~]$ ssh funa@192.168.0.11
 funa@192.168.0.11's password: 
 Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.9.140-tegra aarch64)
 ```
### 3.2 ssh 鍵を使った接続
- 鍵を置くフォルダはhost側client側とも ```~/.ssh```
- 鍵を作る時、秘密鍵にパスフレーズを設定しない。
  (設定すると鍵認証なのに毎回パスフレーズの入力を求められる)
- 公開鍵の登録はクライアント側から行う
  ```
  $ scp .ssh/id_rsa.pub funa@192.168.0.11:./ssh
  $ ssh-copy-id ~/.ssh/id_rsa.pub
  ```
- host側client側とも .ssh フォルダ、鍵ファイルのアクセス権に注意
  - .ssh/ は 700 id_rsa は 600 authorized_keys 600 config 600
  - Permission denied が出る原因はほぼこれ

  
## 4.リモートデスクトップ
- vnc は遅くて使い物にならないため、RDPプロトコルで接続する。
- jetson側で 以下の通り xrdp サーバーをインストールし実行
```
$ sudo apt-get install xrdp
$ sudo systemctl start xrdp
$ sudo systemctl enable xrdp
```

- Client からJetsonの xfce4 デスクトップに接続
```
$ xfreerdp /v:192.168.0.11 /size:"1520x1028" /u:funa /wallpaper /sec-rdp /sound +glyph-cache
```
- xfce4 でターミナルが起動できなくなった時のおまじない。
    ```$ dbus-update-activation-environment --all```

## 5.VSCode から Remote-SSH で Jetson nano にアクセス

## 6.開発環境セットアップ
### 6.1. OpenCV 4.1.2 ビルド
- 基本的にこちらにあるとおり。
  Jetson Nano に OpenCV 4.1.2 をインストールする [https://qiita.com/daisuzu_/items/8cc8de8ea8dc557a2aad]
- ただし前もって BLAS のインストールガ必要
  ```
  sudo apt-get -yV install libopenblas-dev
  sudo apt-get -yV install liblapacke-dev
  ```
- BLAS とはベクトルと行列に関する基本線型代数操作を実行するライブラリAPIのデファクトスタンダードである。オープンソースの最適化 BLAS 実装として OpenBLAS や ATLAS（英語版） がある。

### 6.2. .Net Core インストール
#### 6.2.1 リポジトリからインストール （うまくいかなかった）
- 準備編
```
$ sudo apt update
$ sudo apt upgrade
$ # Microsoft リポジトリを有効化
$ wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
$ sudo dpkg -i packages-microsoft-prod.deb
$ sudo add-apt-repository universe
$ sudo apt update
$ # apt を https 対応するパッケージらしい
$ sudo apt-get install apt-transport-https
```
- インストール
```
$ sudo apt-get install dotnet-sdk-3.1
$ # この場合、/usr/share/dotnet/ にインストールされる。
```
#### 6.2.2 ダウンロードして home 下にインストール
```
mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-3.1.100-linux-x64.tar.gz -C $HOME/dotnet
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet
```
#### 6.2.3 バージョン切り替え
- 同じディレクトリ（$HOME/dotnet）に複数バージョンの .Net Core SDK をインストール可能
- インストールされているSDKのバージョンを確認する
```
$ dotnet --list-sdks
2.2.401 [C:\Program Files\dotnet\sdk]
2.2.402 [C:\Program Files\dotnet\sdk]
3.0.101 [C:\Program Files\dotnet\sdk]
```
- 有効な donet の情報を表示
```
$ dotnet --version  # 現在のバージョン確認
3.1.101
$ dotnet --info     # 詳細情報表示
.NET Core SDK (reflecting any global.json):
 Version:   3.1.101
 Commit:    b377529961

Runtime Environment:
 OS Name:     ubuntu
 OS Version:  18.04
 OS Platform: Linux
 RID:         ubuntu.18.04-arm64
 Base Path:   /home/funa/dotnet/sdk/3.1.101/
(以下略)
``` 
ｰ dotnet プロジェクト作成
```
$ dotnet new ProjectType -n ProjectName -o ProjectFolder
$ # ProjectType 
```

ｰ dotnet のバージョンはプロジェクトフォルダのglobal.jsonで指定する。

#