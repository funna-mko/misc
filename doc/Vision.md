# Jetson を使った映像関係の TIPS
# 1. カメラ関係
## 7.1 カメラ情報
ｰ カメラデバイス ```$ ls -l /dev/video*``` でデバイスが表示される。
- v4l2 のインストール
    v4l-utils パッケージをインストールして v4l2-ctl を使えるようにする。
    ```$ sudo apt-get install v4l-utils```
- デバイスの詳細情報確認
    ```
    $ v4l2-ctl --list-devices    # カメラデバイスの情報表示
    $ v4l2-ctl --list-formats    # 対応可能な画像フォーマット
    $ v4l2-ctl --list-formats-ext
    $ v4l2-ctl --all             # 全部表示
    ```
# 2. カメラ画像のプレビュー
    $ nvgstcapture
    j,q,C^c で終了

# 2. Gstreamer



## 2.1 Gstreamerとは
## 2.2 