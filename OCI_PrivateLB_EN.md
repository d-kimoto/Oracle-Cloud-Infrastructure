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
  - Azure probe probe port monitor resource
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

1. ミラーディスク用のパーティションを作成
   - Linuxの場合
     - /dev/oracleoci/oraclevdb1：RAW
     - /dev/oracleoci/oraclevdb2：ext4でフォーマット
   - Windowsの場合
     - D:\ ：ファイルシステム未作成
     - E:\ ：NTFSでフォーマット
1. CLUSTERPRO をインストールし、ライセンスを登録する
1. Cluster WebUIを起動し、設定モードからクラスタ生成ウィザードを実行する
1. 基本設定、インタコネクトを設定する
   - インタコネクト1
     - Node1：10.0.0.8
     - Node2：10.0.0.9
     - MDC：なし
   - インタコネクト2
     - Node1：10.0.10.8
     - Node2：10.0.10.9
     - MDC：mdc1
1. NP解決を設定する
   - 種類：Ping
   - ターゲット：10.0.0.1
1. フェイルオーバグループを作成する
1. ミラーディスクリソースを作成する
  - Linuxの場合
    - 詳細
      - ミラーパティションデバイス名：/dev/NMP1
      - マウントポイント：/mnt/md1
      - データパーティションデバイス名：/dev/oracleoci/oraclevdb2
      - クラスタパーティションデバイス名：/dev/oracleoci/oraclevdb1
      - ファイルシステム：ext4
  - Windowsの場合	
    - 詳細
      - データパーティションのドライブ文字：E:\
      - クラスタパーティションのドライブ文字：D:\
      - ミラーディスクコネクト：mdc1
      - 起動可能サーバ：Node1, Node2
1. Azure プローブポートリソースを作成する
   - 詳細
     - プローブポート：26001
1. モニタ/監視リソースを作成する
   - 以下のモニタ/監視リソースはフェイルオーバグループのリソース作成時に自動的に作成される
   - Linuxの場合
     - ミラーディスクコネクトモニタリソース
     - ミラーディスクモニタリソース
     - Azure プローブポートモニタリソース
     - Azure ロードバランスモニタリソース
   - Windowsの場合
     - ミラーコネクト監視リソース
     - ミラーディスク監視リソース
     - Azure プローブポート監視リソース
     - Azure ロードバランス監視リソース
1. Cluster WebUI から設定の反映を行う

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
