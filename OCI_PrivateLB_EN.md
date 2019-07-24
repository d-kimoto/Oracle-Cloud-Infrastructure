Oracle Cloud Infrastructure (using internal load balancer)
===

About this guide
---
This guide describes how to setup EXPRESSCLUSTER of the mirror disk type cluster on Oracle Cloud Infrastructure.  
The following describes the cluster configuration using a internal load balancer.  
For the detailed information of EXPRESSCLUSTER X, please refer to this site.()  

Configuration
---
### Overview
In the configuration of this guide, create 2-server(Node1 Node2 as below) cluster of mirror disk type.
If a failure has occurred, the data on block storage is taken over to the other server.
And active and standby servers of the cluster are swiched by controlling the Oracle Cloud Infrastructure load balancer from EXPRESSCLUSTER.
Client Applications can use public IP address to access instance in the virtual cloud network.
if your environment use private IP address, it becomes possible to communicate from Node1 to Node2 and this network use data transfer.

### Software versions
- In the case of Linux
  - Cent OS 6.10 (2.6.32-754.14.2.el6.x86_64)
    or
    Cent OS 7.6 (3.10.0-957.12.2.el7.x86_64)
  - EXPRESSCLUSTER X 4.1 for Linux (internal version：4.1.1-1)
- in the case of Windows
  - Windows Server 2016 Standard
  - EXPRESSCLUSTER X 4.1 for Windows (internal version：12.11)

### Cluster configurations
- network partition resolution resource
  - network partition resolution resource by PING method
- Group resources
  - mirror disk resource
  - Azure probe port resource
- Monitor resource
  - mirror connect monitor resource
  - mirror disk monitor resource
  - Azure probe port monitor resource
  - Azure load balance monitor resource

Oracle Cloud setup
---
1. インスタンスを作成する
   - 拡張オプションでフォルト・ドメインを分ける
     - Node1
        - 可用性ドメイン：可用性ドメイン1 (oIJw:AP-TOKYO-1-AD-1) 
        - フォルト・ドメイン：FAULT-DOMAIN-1
        - パブリックIPアドレス：10.0.0.8
        - プライベートIPアドレス：10.0.10.8
     - Node2
        - 可用性ドメイン：可用性ドメイン1 (oIJw:AP-TOKYO-1-AD-1)
        - フォルト・ドメイン：FAULT-DOMAIN-2
        - パブリックIPアドレス：10.0.0.9
        - プライベートIPアドレス：10.0.10.9
1. ブロック・ボリュームを作成する
   - 2ノード分のブロック・ボリュームを作成
1. インスタンスにブロック・ボリュームをアタッチする
   - デバイス・パス(/dev/oracleoci/oraclevdb)を選択する
   - iSCSIコマンドによりアタッチする
1. ロード・バランサの作成
   - 可視性タイプの選択で「プライベート」を選択
   - バックエンドの追加 をスキップ
   - ヘルス・チェック・ポリシーの指定
     - プロトコル：TCP
     - ポート：26001
     - 間隔(ミリ秒)：5000
     - タイムアウト(ミリ秒)：3000
     - 再試行回数：2
   - リスナーの設定
     - トラフィックのタイプ：TCP
     - リスナーでモニターするポート：80
1. ロード・バランサへのバックエンド追加
   - 先の手順で作成したロード・バランサにクラスタノードのIPを指定してバックエンドを追加する
     - 追加方法：IPアドレス
     - Node1
       - IPアドレス：10.0.10.8
       - ポート：8080
     - Node2
       - IPアドレス：10.0.10.9
       - ポート：8080

Setup EXPRESSCLUSTER X
---
※記載がない値は既定値を設定している

Other parameters than below, default value is setting.

1. Setting the partition for mirror disk
   - In the case of Linux
     - /dev/oracleoci/oraclevdb1：no format (RAW)
     - /dev/oracleoci/oraclevdb2：format ext4
   - In the case of Windows
     - D:\ ：no format
     - E:\ ：format NTFS
1. Install EXPRESSCLUSTER and register license.
1. In Config mode of the Cluster WebUI, executing Cluster generation wizard.
1. setting the Basic Settings and Interconnect.
   - interconnect1
     - Node1：10.0.0.8
     - Node2：10.0.0.9
     - MDC：do not use
   - interconnect2
     - Node1：10.0.10.8
     - Node2：10.0.10.9
     - MDC：mdc1
1. Setting the NP Resolution
   - Type：Ping
   - Ping Target：10.0.0.1
1. Setting the Failover Group
1. Setting the mirror disk resource
  - In the case of Linux
    - Details
      - Mirror Partition Device Name：/dev/NMP1
      - Mount Point：/mnt/md1
      - Data Partition Device Name：/dev/oracleoci/oraclevdb2
      - Cluster Partition Device Name：/dev/oracleoci/oraclevdb1
      - File System：ext4
  - In the case of Windows	
    - Details
      - Date Partition Drive Letter：E:\
      - Cluster Partition Drive Letter：D:\
      - Mirror Disk Connect：mdc1
      - Servers that can run the group：Node1, Node2
1. Setting Azure probe port resource
   - Details
     - Probeport：26001
1. Setting monitor resources
   - The following that monitor resource is automatically registered when setting group resouces.
   - In the case of Linux
     - mirror disk connect monitor resource
     - mirror disk monitor resource
     - Azure probe port monitor resource
     - Azure load balance monitor resource
   - In the case of Windows
     - mirror connect monitor resource
     - mirror disk monitor resource
     - Azure probe port monitor resource
     - Azure load balance monitor resource
1. In Config mode of the Cluster WebUI, executing Apply the Configuration File.

Check the operation for EXPRESSCLUSTER X
---
1. Please look up how to check the operation X in the URL below.

★ もしかしてAzure向けのURL(EN)ない？

参考
---
- CLUSTERPRO X 4.1 Microsoft Azure 向け HA クラスタ 構築ガイド (Linux 版)
   - https://jpn.nec.com/clusterpro/clpx/doc/guide/HOWTO_Azure_X41_Linux_JP_01.pdf
- CLUSTERPRO X 4.1 Microsoft Azure 向け HA クラスタ 構築ガイド (Windows 版)
   - https://jpn.nec.com/clusterpro/clpx/doc/guide/HOWTO_Azure_X41_Windows_JP_01.pdf
