# Linux Tips

# Contents
- 1. OSバージョン確認方法
- 2. パッケージ管理 yum
- 3. 論理ボリューム LVM


## 1. OSバージョン確認方法

```
$cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core) 
```

## 2. パッケージ管理 yum
### 2.1 yum リポジトリ
```
### インストール済み yum リポジトリ確認方法
$ yum repolist [all|enabled|disabled]
読み込んだプラグイン:fastestmirror, langpacks
Determining fastest mirrors
epel/x86_64/metalink                                     | 6.7 kB     00:00     
 * base: ftp.nara.wide.ad.jp
 * epel: sg.fedora.ipserverone.com
 * extras: ftp.nara.wide.ad.jp
 * nux-dextop: mirror.li.nux.ro
 * updates: ftp.nara.wide.ad.jp
base                                                     | 3.6 kB     00:00     
code                                                     | 2.9 kB     00:00     
epel                                                     | 5.4 kB     00:00     
extras                                                   | 3.4 kB     00:00     
google-chrome                                            | 1.3 kB     00:00     
ius                                                      | 1.3 kB     00:00     
nodesource                                               | 2.5 kB     00:00     
  :

```
```
### リポジトリ情報の詳細確認
$ /etc/yum.repos.d/<リポジトリ名>.repo

### 目的のパッケージがインストール済みリポジトリにあるか検索
$ yum search <パッケージ名>

### パッケージインストール時にリポジトリを有効にする
$ yum --enablerepo=<リポジトリ名> install <パッケージ名>

```

## 3. 論議ボリューム LVM
### 3.1 LVM 概要
LVM (Logical Volume Manager) は物理ボリュームに直接ファイルシステムを構築せずに、一旦、論理ボリュームを構築し、その上にファイルシステムを構築することによって、複数物理ボリュームを跨るファイルシステムを構築できるようにする仕組み。

**extent**
LVM 上の区切り単位（物理デバイスのブロックのようなもの）で物理ボリューム(PV)に割り当てられる物理exetent と論理ボリューム(LV)上に割り当てられる論理extent に分けられる。論理extent には1つ以上の物理extent が割り当てられる。（2つ以上割り当てた場合はミラーリングとして作用する。
1 extent はデフォルトで 4 MiB 

**ボリュームグループ VG**
物理ボリューム(PV)上のエクステントをひと纏めにした単位。VG上に複数の論理ボリューム(LV)を構築できる。（LVは物理デバイス上のパーティションに相当）


## 4.OS関連
### 4.1 カーネルパラメータの読み書き (sysctl)
ネルパラメータは /proc/sys 配下にファイルとしてリストされている。また、システム起動時 /etc/sysctl.conf からロードして設定される。

確認、設定は次の通り。
 - sysctl -d \<param-name>  // パラメータ値を確認する。
 - sysctl -w \<param-name>=\<value> // 値設定

### 4.2 サービスの管理 (systemctl)

以下のコマンドでサービスの状態確認|開始｜終了｜自動起動する｜しない を設定する。
> systemctl (status|start|stop|enable|disable) \<service-name> 

- /etc/systemd/Xxxx.service // 各サービスの設定ファイル
- systemctl list-units -type=service  // 稼働中のサービス一覧
- systemctl list-unit-files --type=service // 定義されたサービス一覧
- systemctl (is-active|is-enabled|is-failed) <service> // サービス個別の状態確認

### 4.2 Daemon 起動確認