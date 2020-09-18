# ネットワーク設定の基本
# NetworkManager使い方
- デバイス確認
```
nmcli d show
nmcli c show
```
- 設定確認
```
> nmcli c show <ConName>
> # リストが長くて見難い時、表示するフィールド名を限定できる。
> nmcli -f ipv4 c sh <ConName>  # 例）ipv4 で始まる項目だけ表示
```
- IPアドレス設定
```
> sudo nmcli c mod <ConName> ipv4.method manual ipv4.address "192.168.0.33/24" ipv4.gateway 192.168.0.1
# 個別に指定しても良い。
# 設定を削除する時は -ipv4.address の様に項目名にハイフンをつける。
# gateway を無効化する時は -ipv4.gateway ではなく 0.0.0.0 を指定する。
# ipv4.never-default yes になっていると gateway が設定できない。
```
- DNS設定
```
> sudo nmcli c mod <ConName> ipv4.dns 192.168.0.1
```
- 設定変更の反映
```
> sudo nmcli c reload                # 設定のリロード
> sudo nmcli c down <ConName>        # コネクション無効化
> sudo nmcli c up <ConName>          # コネクション有効化
```
# システム設定
- NetworkManager 停止／起動／再起動／状態確認
```
> systemctl stop NetworkManager
> systemctl start NetworkManager
> systemctl restart NetworkManager
> systemctl status NetworkManager
```
- ネットワーク設定ファイル
```
> # nmcli con mod <ConName> で指定した設定が以下のファイルに反映される。
> cat /etc/sysconfig/network-script/ifcfg-<ConName>
```
- カーネルパラメータ
IPフォワーディングと逆方向パス転送を設定する。
  - IPフォワーディング
  net.ipv4.ip_forward 複数NICがあるPCでNICを超たパケット転送を有効にする。
  - 逆方向パス転送
  rp_filter (Reverse Path Filtering) 複数NICを実装するサーバーでパケット入力元NICと \
  送信先NICが異なる場合にフィルタリングする設定。
    0 — フィルタしない
    1 — RFC 3704 で定義された厳密なモード。
    2 — RFC 3704 で定義された緩慢なモード。 
  
```
> cat /proc/sys/net/ipv4/ip_forward         # IPフォワーディングの設定確認
0
> sudo echo 1 > /proc/sys/net/ipv4/ip_forward   # 一時的に設定
> cat /proc/sys/net/ipv4/conf/all/rp_filter     # 逆方向パス転送
1
> sudo echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter  # 一時的に無効化
> vi /etc/sysctl.conf      # 恒久的に変更するにはこのファイルに書いてネットワーク再起動
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
...
> cat /etc/sysctl.conf
```

## NIC2枚で異るセグメントを接続する設定
下図のトポロジーで 192.168.3.0 のネットワークから Wifiルータを経由して
インターネットにアクセスするための、PC1,Jetsonの設定を示す。
```
    Wifi Rooter <192.168.0.1>
      |
      |  192.168.0.0/24
      |
    wlp1s0 Bafalo-G-3A38 <192.168.0.13>
    PC-1
    eno1 eno1            <192.168.3.1>
      |
      |  192.168.3.0/24
      |
    eth0 "Wired connection 1" <192.168.3.2>
    Jetson
```
- Wifiルーターの設定
  特になし

- PC1の設定（全体）
  - IPフォワーディング設定
     echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
  - 逆方向パス転送フィルタ無効化
     echo net.ipv4.conf.all.rp_filter=0 >> /etc/sysctl.conf
- PC1の設定（Wifi側）
  - /etc/sysconfig/network-script/ifcfg-Bufalo-G-3A38
    ONBOOT=yes
    BOOTPROTO=dhcp
  - nmcli c s Bufalo-G-3A38
    ipv4.method:  auto
    IP4.ADDRESS[1]: 192.168.0.13/24
    IP4.GATEWAY:    192.168.0.1
    IP4.DNS:        192.168.0.1
  - IPマスカレード有効
    firewalld-cmd --zone=public --add-masquerade --permanent
    firewalld-cmd --reload
- PC1の設定（LAN側）
  - /etc/sysconfig/network-script/ifcfg-eno1
    ONBOOT=yes
    BOOTPROTO=none
    ZONE=trusted
  - nmcli c s eno1
    ipv4.method:  manual
    IP4.ADDRESS[1]: 192.168.3.1/24
    IP4.GATEWAY:    --
    IP4.DNS:        --
  - IPマスカレード有効
    firewalld-cmd --zone=trusted --add-masquerade --permanent
    firewalld-cmd --reload
- Jetsonの設定
  - nmcli c s eth1
    ipv4.method:  manual
    IP4.ADDRESS[1]: 192.168.3.2/24
    IP4.GATEWAY:    192.168.3.1
    IP4.DNS:        192.168.0.1



```
# システム設定
net.ipv4.ip_forward=1         # IPフォワーディング有効
net.ipv4.conf.all.rp_filter=0 # 逆方向パスフィルタ無効
# IF設定

```

## firewalld 設定
コンピュータAにネットワークデバイス(eth0,wifi0,等)が2つ以上ある場合、片側のデバイスを gateway 経由でインターネットへ、他のデバイスを別のコンピュータに接続するようなケースで行う設定。
（以前のバージョンでは iptables で設定していた）

「firewalld」で通信制御のポリシー設定は、事前に定義されたゾーンに対して通信の許可・遮断ルールを適用し、そのゾーンを各NIC（ネットワークアダプタ）に割り当てていく方式で行う。 

- サービスの起動状態を表示
``` 
# firewall-cmd --state      // sysctrl firewalld と同じ
# firewall-cmd --list-all   // デフォルトゾーンの設定を表示
# firewall-cmd --zone=<ZoneName> --list-all // Zone指定
# firewall-cmd --list-all-zones  // 全Zoneを一気に表示
``` 
- Zone 確認／切り替え
```
> firewall-cmd --get-active-zones    # ActiveZone表示
public
  interfaces: wlp1s0
trusted
  interfaces: eno1
> # インターフェースのZone切り替え
> firewall-cmd --zone=home --change-interface=eno1 --permanent
> firewall-cmd --reload

```
- Zone へサービスを追加
```
# firewall-cmd --zone=public --add-service=http
success    ←一時的に追加されただけ
# firewall-cmd --permanent --zone=public --add-service=http
success
# firewall-cmd --reload
success    ←一恒久的に追加された
```
## 静的ルート設定
nmcli c mod eno1 +ipv4.routes "192.168.0.0/24 192.168.0.13"
(before)
default via 192.168.0.1 dev wlp1s0 proto dhcp metric 600 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.0.0/24 dev wlp1s0 proto kernel scope link src 192.168.0.13 metric 600 
192.168.3.0/24 dev eno1 proto kernel scope link src 192.168.3.1 metric 100 
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 
(after)
default via 192.168.0.1 dev wlp1s0 proto dhcp metric 600 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.0.0/24 via 192.168.0.13 dev eno1 proto static metric 100 
192.168.0.0/24 dev wlp1s0 proto kernel scope link src 192.168.0.13 metric 600 
192.168.0.13 dev eno1 proto static scope link metric 100 
192.168.3.0/24 dev eno1 proto kernel scope link src 192.168.3.1 metric 100 
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 

# ブリッジ接続
## KVM内にブリッジを作成し、仮想マシンをWifi経由でインターネットに接続する。
- パケット転送を有効にする。
  ```# sysctl -w net.ipv4.ip_forward=1 ```
  再起動後も有効にするには ``` /etc/sysctl.conf``` 内に ```net.ipv4.ip_forward = 1```と書き込む。
  確認方法は
  ```
  # sysctl net.ipv4.ip_forward
  net.ipv4.ip_forward=1
  ```

- ブリッジの作成
  ```
  # nmcli con add type bridge ifname br0 stp no
  接続 'bridge-br0' (75e3960c-f2b2-49ff-b7d5-69102fb8b1cb) が正常に追加されました。
  ```

- ネットワークの再起動
   ```# systemctl restart network```
### 初期状態
```
[mikio@localhost ~]$ nmcli d
DEVICE      TYPE      STATE     CONNECTION     
wlp1s0      wifi      接続済み  Buffalo-G-3A38 
virbr0      bridge    接続済み  virbr0         
eno1        ethernet  利用不可  --             
lo          loopback  管理無し  --             
virbr0-nic  tun       管理無し  --             
[mikio@localhost ~]$ 
[mikio@localhost ~]$ nmcli c
NAME              UUID                                  TYPE      DEVICE 
Buffalo-G-3A38    df0ec271-d117-419e-b298-980b229753c8  wifi      wlp1s0 
virbr0            d3ae459b-4716-4ec5-b282-59d7c5debccc  bridge    virbr0 
Buffalo-A-3A38    0e390f4e-3e82-489a-8722-40c9fbe03acc  wifi      --     
Buffalo-A-3A38 1  49286743-0d59-4cff-b60d-215e144d6817  wifi      --     
eno1              ca75f540-5b98-492c-a7fc-629d760cdd74  ethernet  --     
wlp1s0-br0        cf1a170e-52a9-477b-bc8e-73631957a6bd  ethernet  --     
[mikio@localhost ~]$ 
[mikio@localhost ~]$ ls /etc/sysconfig/network-scripts/ifcfg*
/etc/sysconfig/network-scripts/ifcfg-Buffalo-A-3A38
/etc/sysconfig/network-scripts/ifcfg-Buffalo-A-3A38_1
/etc/sysconfig/network-scripts/ifcfg-Buffalo-G-3A38
/etc/sysconfig/network-scripts/ifcfg-eno1
/etc/sysconfig/network-scripts/ifcfg-lo
/etc/sysconfig/network-scripts/ifcfg-wlp1s0-br0
[mikio@localhost ~]$ 
```

- ブリッジデバイスを作成する
- ブリッジの設定
- ブリッジをネットワークに接続


## parprouted


cmake 
-D WITH_CUDA=ON 
-D CUDA_ARCH_BIN="5.3" 
-D CUDA_ARCH_PTX="" 
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.0/modules 
-D WITH_GSTREAMER=ON 
-D WITH_LIBV4L=ON 
-D BUILD_opencv_python2=ON 
-D BUILD_opencv_python3=ON 
-D BUILD_TESTS=OFF 
-D BUILD_PERF_TESTS=OFF 
-D BUILD_EXAMPLES=OFF 
-D CMAKE_BUILD_TYPE=RELEASE 
-D CMAKE_INSTALL_PREFIX=/usr/local ..

cmake -D CUDA_HOST_COMPILER=/usr/bin/gcc-7 
-D CUDA_NVCC_FLAGS="--expt-relaxed-constexpr" 
-D CUDA_VERSION="10000" 
-DWITH_NVCUVID=OFF 
-DENABLE_CCACHE=OFF 
-DEIGEN_INCLUDE_PATH=/usr/include/eigen3 
-D HAVE_CUDA=1 
-D WITH_CUDA=ON 
-D CUDA_ARCH_BIN="5.3" -D CUDA_ARCH_PTX="" -DWITH_FAST_MATH=OFF -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.1/modules -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON 
-D BUILD_opencv_python2=OFF 
-D BUILD_opencv_python3=ON 
-DPYTHON3_EXECUTABLE=/usr/bin/python3 -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local/ -D WITH_CUBLAS=ON -D LAPACK_LIBRARIES=/usr/lib/aarch64-linux-gnu/libblas.so -D LAPACK_CBLAS_H=/usr/include/aarch64-linux-gnu/cblas.h ..


cmake 
-D CUDA_HOST_COMPILER=/usr/bin/gcc-7 
-D CUDA_NVCC_FLAGS="--expt-relaxed-constexpr" 
-D CUDA_VERSION="10000" 
-D WITH_NVCUVID=OFF 
-D ENABLE_CCACHE=OFF 
-D EIGEN_INCLUDE_PATH=/usr/include/eigen3 
-D HAVE_CUDA=1 
-D WITH_CUDA=ON @
-D CUDA_ARCH_BIN="5.3" @
-D CUDA_ARCH_PTX="" @
-D WITH_FAST_MATH=OFF 
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.1.2/modules *
-D WITH_GSTREAMER=ON @
-D WITH_LIBV4L=ON @
-D BUILD_opencv_python2=OFF *
-D BUILD_opencv_python3=ON @
-D PYTHON3_EXECUTABLE=/usr/bin/python3 
-D BUILD_TESTS=OFF @
-D BUILD_PERF_TESTS=OFF @
-D BUILD_EXAMPLES=OFF @
-D CMAKE_BUILD_TYPE=RELEASE @
-D CMAKE_INSTALL_PREFIX=/usr/local/ @
-D WITH_CUBLAS=ON 
-D LAPACK_LIBRARIES=/usr/lib/aarch64-linux-gnu/libblas.so 
-D LAPACK_CBLAS_H=/usr/include/aarch64-linux-gnu/cblas.h ..



