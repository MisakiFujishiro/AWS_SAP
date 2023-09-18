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

### インターフェース型(PrivateLink型)
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





## 外部からVPCへの接続
AWS外部からVPCへの接続を行う方法について、大きく3パターン
- AWS クライアントVPN: クライアントとの接続設定
- AWS Site-to-Site VPN: データセンタやオフィスとの接続設定
- Direct Connect: 物理線を引いての接続設定

### AWS クライアントVPN
VPCの外部のクライアントから、VPCに接続する技術

![](../img/chap1_vpn_client.png)

#### 設定方法
1. クライアントが利用するCIDRを設定
2. 認証タイプを設定

![](../img/chap1_vpn_setting_1.png)

3. サブネットにVPNを関連づける  
    VPNのENIが発行される  
    VPNを経由して、別のAWSリソースにアクセスする際はこのENIのIPが利用される
4. 認証設定  
    デフォルトでは、VPNない全てのリソースにアクセス権限を持っていないので、明示的にアクセス先を許可する

![](../img/chap1_vpn_setting_2.png)

5. クライアント用のVPN Endpoint設定をDLする  
    作成されたエンドポイント情報をDLして、クライアント側でVPNの設定を行う

#### 認証タイプ
AWSクライアントがVPNに接続する際の認証は以下の3パターンが準備されている
- Active Directory認証(ユーザーベース)
- シングルサインオン：SAMLベースのフェデレーション(ユーザーベース)
- 相互認証（証明書ベース）



### AWS Site-to-Site VPN
VPCに仮想プライベートゲートウェイをアタッチして、データセンターなどのオンプレのルーターと、IPsec VPN接続を可能にする。
自動的に2つの接続が設定され、高可用性が担保される。

![](../img/chap1_vpn_site2site.png)


### 仮想プライベートGW
VPCエンドポイント側に設定するGW

#### 設定方法
1. VPCの画面から仮想プライベートGWを選択
2. 名前とASNの設定
3. 対象のVPCにアタッチ
4. ルートテーブルに対して、仮想プライベートGWへのルーティングを設定


### カスタマーGW
カスタマーのルーター側で設定するGW

#### 設定方法
1. VPCの画面からカスタマーGWを選択
2. 名前、ルーティングタイプ、ASN、ルーターのIPを設定


### VPN接続の確立
#### 設定方法
1. VPCの画面からSite-to-site VPN接続を選択
2. 仮想プライベートGWとカスタマーGWを選択
3. ルーター側に導入するconfigファイルをDLして、ルーターに導入する


### Direct Connect
ユーザーのルーター（Customer Router）からDirect Connectのルータ（DX Router）に光ファイバーケーブルを介して接続するサービス。インターネットを経由しない専用線を確保できる。

通信速度に関しても最大100GBpsを確保することができるので、金額と設定の手間がかかる分、安全性と通信速度を確保できる。

![](../img/chap1_vpn_direct.png)

接続方法などは[参考サイト](https://atbex.attokyo.co.jp/blog/detail/85/)参照。





## VPC同士の接続
VPCが他のVPCと接続するための設定。  
一対一の場合は、VPCピア接続が容易。一対多の場合はTransit GWを利用すると管理が容易になる。

### VPCピア接続
異なるリージョンであっても、設置を行うことでVPC-AとVPC-Bで通信を可能にする。
ただし、VPC-AとVPC-BがVPCピア接続していても、VPC-Bとピア接続しているVPC-CからVPC-Aへの接続はできない。

![](../img/chap1_vpc_peer_image.png)

#### 設定方法
[参考サイト](https://zenn.dev/rescuenow/articles/8e21a751c828c6)がわかりやすいので確認すると良い。

1. リクエスタ(接続元)側でピア接続設定を行う
    - 接続元のVPC ID
    - 接続先のアカウント情報（同じアカウントか否か）
    - 接続先のリージョン（同一リージョンか否か）
    - 接続先のVPC ID
![](../img/chap1_vpc_peer_setting_1.png)
2. アクセプタ(接続先)側で承認する
    - ピア接続設定を確認すると承認待ちになっているので承認をする
3. リクエスタとアクセプタ両方でルートテーブルの設定を行う
    - CIDRの設定を行う
    - ターゲットにピアリング接続を行う
![](../img/chap1_vpc_peer_setting_2.png)



### Transit Gateway
最大で5000のVPC同士の接続やVPCとオンプレとの接続を管理することができる。
Site-to-Site VPN接続やVPCピア接続について個別設定せずに、Transit GWで集中管理できる。

![](../img/chap1_transitgw_image.png)

#### VPCピア接続の取りまとめ
VPCピア接続を利用して複数のVPC同士を接続すると以下のような課題がある
- トランジットトラフィックに対応できない
    - VPCを跨いだ通信はできない
    - すなわち、すべてのVPC同士の設定が必要
- VPCを追加すると、全てに修正が入る

Transit GWを利用することで、トランジットトラフィックに対応でき、追加時もTransitGWを修正するだけで良い

![](../img/chap1_transitgw_merit_1.png)


#### VPN接続の取りまとめ
VPN接続を利用して複数のVPCに接続しようとすると、それぞれのVPCに対してVPNの設定が必要。
しかし、TransitGWを利用すれば、オンプレとTransit GWまでの設定をするだけで、後ろのVPCの追加が容易になる。

![](../img/chap1_transitgw_merit_2.png)





## Route53
### HosteZone
パブリックホストゾーンとプライベートホストゾーンがある。
- パブリック：インターネット上に公開されたDNSドメインを管理
- プライベート：VPC内のドメインのレコードを管理

![](../img/chap1_route53_hostzone.png)

### Resolver
VPC内やオンプレからの問い合わせを解決するサービスで[こちらの記事](https://dev.classmethod.jp/articles/explain-route53-resolver-ram/)がわかりやすい

#### VPC内の名前解決
通信を行う際に、Resolverに問い合わせを行い、Resolverはhosted zoneに問い合わせを行い解決する。

![](../img/chap1_route53_resolver_1.png)

#### オンプレからの名前解決
オンプレミス環境からクラウド環境に名前解決のリクエストをする際には、Route53 Resolverのインバウンドエンドポイントの設定が必要になる。インバウンドのエンドポイントが作成されると、IPが発行されるので、このIPをオンプレ側のDNSに設定する。

![](../img/chap1_route53_resolver_2.png)

#### オンプレへの名前解決
クラウド側からオンプレミスのDNSで名前解決をリクエストする際には、Route53 アウトバウンドエンドポイントとRoute53 ResolverRuleを設定する必要がある。

このResolver Ruleは共有が可能で、別のAWSアカウントなどで共通利用することができる。

![](../img/chap1_route53_resolver_3.png)

