# ネットワーク設計
VPCエンドポイントや、VPN接続などが問われることが多いので、以下を確認しておくと良い
- [VPCのBlackBelt資料](https://d1.awsstatic.com/webinars/jp/pdf/services/20201021_AWS-BlackBelt-VPC.pdf)
- [VPCのBlackBelt動画](https://aws.amazon.com/jp/blogs/news/webinar-bb-amazonvpc-2020/)
- [VPC AdvancedのBlackBelt資料](https://www.cloudremix.net/awsdocs/20190417_AWS-BlackBelt-VPC-Advanced.pdf)
- [VPC AdvancedのBlackBelt動画](https://aws.amazon.com/jp/blogs/news/webinar-bb-amazon-vpc-advanced-2019/)


## VPCエンドポイント
AWSのサービス（パブリックネットワークに存在）に対してアクセスする際、インターネットを経由せずにAWS内部の通信で接続するために利用されるのがVPCエンドポイントである。

VPCエンドポイントにはゲートウェイ型とインターフェース型が存在する

### ゲートウェイ型
S3とDynamo DBが対応しており、料金がかからない特徴がある。  
ゲートウェイであるので、インターネットGWなどと同様、ルーティング設定された条件を満たすとVPCエンドポイントを経由して、リソースにアクセスする。

![](../img/chap1_vpc_endpoint_gw.png)

作成としては以下の手順を踏む
1. VPCサービスのエンドポイント機能を選択
2. com.amazonaws.ap-northeast-1.s3のGWを選択
3. 設定するVPCを選択
4. 設定する`ルートテーブル`を選択
5. バケットポリシー側で設定したVPC-Endpointからのアクセスを許可する

![](../img/chap1_vpc_gw_setting_1.png)
![](../img/chap1_vpc_gw_setting_2.png)

### インターフェース型
50種類以上のサービスが対応しており、S3はインターフェース型の接続も可能。  
データの転送とサービス利用時間に応じて課金する必要がある。  
VPC内のサブネットにENIを払い出し、ENIにアクセスすることで対象のAWSサービスにアクセスする。
AWS PrivateLinkという技術が利用される。


![](../img/chap1_vpc_endpoint_if.png)

作成としては以下の手順を踏む
1. VPCサービスのエンドポイント機能を選択
2. com.amazonaws.ap-northeast-1.s3のInterfaceを選択
3. 設定するVPCを選択
4. 設定する`サブネット`を選択
5. 適用する`セキュリティグループ`を選択
6. バケットポリシー側で設定したVPC-Endpointからのアクセスを許可する

![](../img/chap1_vpc_if_setting_1.png)
![](../img/chap1_vpc_if_setting_2.png)

#### サードパーティサービスの提供
自作のサービス（サードパーティサービス）を他のVPCやオンプレに対してプライベートに利用してほしい場合はNLBを経由して、インターフェース型のVPCエンドポイントを払い出すことができる。

![](../img/chap1_vpc_therdparty_image.png)


### ゲートウェイ型とインターフェース型の違い
ゲートウェイ型のEndpointのIPは、内部IPではなく外部IPであるという点が特徴的である。
すなわちAWS内部からAWS内部への通信であるものの、VPC内→エンドポイントへの通信は外部のIPへのアクセスに見える。  
一方で、インターフェース型はENIがVPCの内部にサービスが立ち上がっているように見えるので、VPC内部→VPC内部の通信で完結する。  
この辺りは[2つのVPCエンドポイントの違いを知る](https://dev.classmethod.jp/articles/vpc-endpoint-gateway-type/)が参考になる。

さらに、VPC外からの接続について差分がある。  
GW型では通信を設定し合ったVPCのみで通信が可能であり、そのVPCと接続している他の環境（オンプレや別のVPC）から Endpointを経由することはできない。  
一方で、インターフェース型は、VPC外からもVPC自体への接続を確保してしまえば、 Endpointを経由して対象サービスにアクセスすることができる。


## VPN



## Direct Connect



## VPCピア接続



## Transit Gateway



## Route53


