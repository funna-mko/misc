# ブリッジ接続

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
