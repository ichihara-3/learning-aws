# Leaning MEMO

## Udemy Ultimate AWS Certified Developer Associate 2020

My Goal::
- Finish ALL the programs by July 31.
- Pass the exam by August.


### IAM

- IAM (Identity and Access Management)
- AWSのセキュリティ管理が行われる。
- Rootアカウントは使われるべきでないし、シェアされてはいけない
- IAMには3つのパートがある
  - Users
  - Groups
  - Roles
- IAMはGlobalな設定

Users
"physical users": 個別の人に割り当てられるアカウント

Groups
ユーザの集合。グループに設定された権限はユーザに継承される

Roles
システムが使う権限


IAM policyはJSONで記述される

IAM Federation
- 大きな企業では企業の認証情報を使ってログインすることができる

注意

- IAM クレデンシャルを絶対にコードに書いてはいけない
- 初期設定以外でルートアカウントを使ってはいない
- ルートのクレデンシャルを使ってはいけない。漏れたら大変。ググれ


### EC2

- AWS で最も基本的で、最も利用されているサービス
- 以下の要素で主に構成される
  - EC2 : 仮想マシンのレンタル
  - EBS : 仮想デバイスへのデータの保存
  - ELB : マシンの負荷分散をする
  - ASG : サービスのオートスケール

#### EC2のインスタンス作成

インスタンス一覧で特定のインスタンスを指定して右クリック→Launch More Like This をクリックすると、そのインスタンスの起動時と同じ設定で新しいインスタンスを作れる

#### tenancy

  default: 通常のインスタンス
  Dedicated instance: ハードウェア占有インスタンス
  Dedicated Host: ハードウェア占有ホスト
https://dev.classmethod.jp/articles/ec2-tenancy/ が参考になる


#### User Data

EC2の初回起動時の処理(インストール・設定など) をスクリプトで定義できる。これにより、初回起動でのセットアップを自動化できる
インスタンス作成時に指定可能
STOP状態のEC2 User DataはコンソールのActionから変更可能だが、編集しても起動時には再実行されない。

#### IP

- Public IP: 公開IP. sshやhttpで繋ぐ時などはこちらのIPをつかう
- Private IP: プライベートIP: AWS内のネットワーク上のIP
- Elastic IP: 固定IP. 所持している間課金される。任意のインスタンスにアタッチ・デタッチできる


#### Security Group

インスタンスやELBなどEC2のサービスにアタッチするinbound/outboundのルール
ip, CIDR, 他のセキュリティグループを元に通信の許可を定義できる
他のセキュリティグループをSourcesに設定したい場合は、Sourcesに `sg-` と打ち込むと候補が出てくるので便利
TCP, HTTPなどのプロトコルとポートを指定することができる

セキュリティグループは任意の数のインスタンスにアタッチでき、インスタンスは任意の数のセキュリティグループをもつことができる

#### ENI Elastic Network Interface

VPCで利用する仮想のネットワークカード

次のような特徴がある

- primary private IP v4
- 一つ以上のsecondary private IP v4 アドレス
- 一つのprivate IP v4 につき一つのElastic IP が付与できる
- 一つのpublic IP V4
- 一つ以上のセキュリティグループをアタッチできる
- MACアドレスを割り当てられる
- インスタンスタイプにより割り当てられる枚数が変わるので注意
- 特定のAZにbound される
- あるインスタンスからデタッチし、他のインスタンスに付け替えることができる

NTT東日本の解説コラム: https://business.ntt-east.co.jp/content/cloudsolution/column-14.html


### Elastic Load Balancer

トラフィックを複数のEC2インスタンスやLambdaなどに振り分けるためのサービス。

ロードバランサは以下のような目的で使う


- ロードを複数のインスタンスに分散させる
- 単一のアクセス点(DNS)をアプリケーションに設定する
- 下流のインスタンスの問題をシームレスに扱う
- 各インスタンスのヘルスチェックを行う
- webサイトのSSL終端を行う
- スティッキーセッションCookieを設定する(後述)
- 複数のAZにまたがってロードバランシングする
- publicなトラフィックをprivateなネットワークに流す

ELBはマネージドなサービス。

#### Health Checks

port / route を定義し、ELBからそのポイントにリクエストを送ることでレスポンスから正常性を判断する
ELBがインスタンスがhealthy であるかを判断し、ロードバランシングの対象にするかどうかを決めるための仕組み


#### Classic Load Balancer

一番古いロードバランサー。(L7)HTTP, HTTPS, (L4)TCP のロードバランシングができる。
現在は後続のApplication Load Balancer / Network Load Balancer が推奨

Health Checks は TCPベースまたはHTTPベースで定義する
固定のホスト名が付与される。xxx.region.elb.amazonaws.com 例) testelb.ap-northeast-1.elb.amazonaws.com


#### Application Load Balancer (v2)

L7向けロードバランサー。HTTP, HTTPS, WebSocketをサポートする。マイクロサービスやコンテナベースのアプリケーションに適している(例: Docker + Amazon ECS)

##### ALBの特徴

- 複数のマシンにまたがったHTTPアプリケーションのロードバランシングを行うことができる。target group
- コンテナのような、同一マシン上の複数アプリケーションにロードバランシングすることもできる
- HTTP/2, WebSocketをサポートする。
HTTP -> HTTPSのようなリダイレクトを設定できる
- ECSのダイナミックポートにリダイレクトするためのポートマッピング機能がある
- ルーティングの方法

  - URL pathベースのルーティング (/users , /posts)
  - hostname ベースのルーティング(one.example.com , other.example.com)
  - クエリストリング、ヘッダベースのルーティング (example.com/users?id=123&order=false)
- 一つのALBで複数のアプリケーションのロードバランシングができる。CLBではアプリケーションごとに一つ設定する必要があった

##### Target Groups ターゲットグループ

ALBのルーティング対象のグループのこと。ルーティング対象やプロトコル、ヘルスチェックなどを定義する。複数作成可能。

ターゲットグループの対象になるものは以下

- EC2インスタンス(Auto Scaling Group で管理可能) - HTTP
- ECSタスク - HTTP
- AWS Lambda - HTTP -> JSON Eventに変換
- IP アドレス- (private IP)

ALBは複数の ターゲットグループを持つことができる。

##### その他のポイント

- ホスト名は固定(xxx.region.elb.amazonaws.com)
- アプリケーションサーバからはクライアントIPは見えない
  - クライアントIPは X-Forwarded-For ヘッダに設定される
  - クライアントのポートは X-Forwarded-Port
  - クライアントのプロトコルは X-Forwarded-Proto
- ターゲットグループの対象インスタンスを設定するためには、インスタンスは起動している必要がある
- ターゲットグループにインスタンスを追加するにはAdd to registered をクリックする必要がある。チェックボックスをつけてSaveするだけでは追加されなかった
- ALBのエンドポイントにアクセスしても繋がらずタイムアウトしそうなときはセキュリティグループの設定を疑うこと


#### Network Load Balancer (v2)

L4向けロードバランサー。TCP, TLS, UDP をサポートする

数百万(millions)オーダーのリクエストを毎秒捌くことができる。
レイテンシは100ms未満程度(ALBは400ms程度)
NLBはAZごとに静的IPをもつ。また、ElasticIPを設定することができる。


##### NLBのユースケース

- 高パフォーマンスが要求される
- TCP / UDPレベルでのトラフィックを扱う必要がある


##### その他のポイント

- NLBにはセキュリティグループがない
- 作成したNLB経由でのアクセスがタイムアウトしていると思われる時は、ターゲットグループのターゲットタブから、Statusがどうなっているか見る。unhealthyである場合、インスタンス側のセキュリティグループに問題がないかみる
- NLBはLayer 4 でのロードバランシングなので、各インスタンスからインターネットのIPがそのまま見える。そのため、インスタンス側のセキュリティグループでインターネットからのアクセスを許可する必要がある。例えばTCP 80ポートのインバウンドセキュリティグループをanywareに開放する必要がある。
- スティッキーセッションが利用できる。source_ipごとに振り分けるインスタンスを固定することでセッションを継続できるとのこと. 参考) https://dev.classmethod.jp/articles/nlb-support-sicky-session/

#### スティッキーセッション Load Balancer Stickiness

同じクライアントのリクエストを常に同じインスタンスにリダイレクトさせる機能。CLB, ALBでサポートされている。
スティッキーセッションの実現にはCookieが利用されている。
ユースケース: セッション上のデータ保持しておきたい場合など
スティッキーセッションを有効にした場合、バックエンドへの負荷分散が偏ることになる

設定方法
CLBではロードバランサレベルで、ALBではターゲットグループレベルで設定できる。デフォルトではオフになっている。
`Edit attributes` から設定を変更することができる。

#### Cross-Zone Load Balancing

AZをまたいでロードバランシングすることができる。

##### CLB

デフォルトではオフになっている。
オンにしても追加料金はかからない

##### ALB

常にオンで、停止できない
オンにしても追加料金はかからない

##### NLB
デフォルトではオフになっている
オンにするとAZ跨ぎのときに追加料金がかかる


#### SSL/TLS in ELB

LBはSSL/TLS通信を終端することができる。LBと背後のインスタンスはHTTPで通信が可能
ロードバランサーはX.509 証明書を利用する。
ACM(Amazon Certificate Manager) によって設定管理をおっこなうことができる。Amazonの提供する証明書だけでなく、独自の証明書も利用することが可能。
TLSを通信を行う場合は、HTTPSのリスナーにデフォルトの証明書を設定する必要がある。複数ドメインの場合はついあの証明書を設定することも可能。

##### SNI (Server Name Indication)

クライアントが接続したいホスト名を伝えることで、一つのWeb サーバーが複数の証明書を使い分けることを可能にする技術のこと。名前ベースのバーチャルホストをHTTPSに対応させたい場合に利用できる
[Server Name Indicaion (Wikipedia)](https://ja.wikipedia.org/wiki/Server_Name_Indication)
古いクライアントはサポートしていない。IE6など

サーバ側は、クライアントから示されたホスト名によって証明書を使い分け、その証明書によってTLSハンドシェイクを試みることができる。

AWSのサービス側でサポートしているのは、ALB, NLB, Cloudfront

CLBでは動作しないので注意が必要。


#### Connection Draining

インスタンスがLBの対象から除外されたりunhealthyになったときに、新規のリクエストは受け付けなくなるが、既に受け付けているリクエストは処理させたい。このときの待ち時間をConnection Drainingと呼ぶ。
デフォルトは300秒で、1秒から3600秒(1時間) までの範囲で設定が可能。
除外対象のインスタンスはConnection Draining の時間がすぎた後に終了処理が行われる。

サービス上では呼び名が違うので注意
CLB: Connection Draining
Target Group(ALB/NLB): Deregistration Delay


#### Auto Scaling Group (ASG)

EC2インスタンスのスケールイン(台数増)・スケールアウト(台数減)を自動で行うための設定。
初期のインスタンス数・最小値・最大値を設定することができ、CloudWatchなどのメトリクスを指標として、自動でスケールイン・スケールアウトをさせることができる。
ASGにより新規につくられたインスタンスは、自動でロードバランサーに追加される。

ASGには以下のような設定項目がある

- 起動設定
  - AMI + Instance Type
  - EC2 User Date
  - EBSボリューム設定
  - セキュリティグループ
  - SSHキーペア
- 最小サイズ・最大サイズ・初期キャパシティ
- ネットワーク・サブネット情報
- ロードバランサー
- スケールアウト・スケールインの基準を決めるポリシー

##### ASGの仕様

ASGはLaunch configuration、またはより新しいLaunch Templates を設定することで定義する。 ASGにアタッチされたIAM Roleは配下のEC2インスタンスに適用される。
Launch Templateでは利用するAMIやインスタンスタイプ、ユーザーデータなどを設定することができる。
Launch configuration ではひとつのインスタンスタイプしか選べないが、Launch TemplateではSPOT Fleetも選択できるほか、オンデマンドとスポットの組み合わせも選択できる。

ASG自体の利用料金は無料。起動されたリソースに対してのみ課金される。

ASGの対象のインスタンスがterminateされたり、unhealthyになった場合は、速やかに新しいインスタンスが自動で作成される。停止や再起動は実行されず、古いインスタンスは自動で破棄される。
ヘルスチェックには２種類ある。1つめはEC2ヘルスチェックで、EC2インスタンスが壊れた場合に自動的にインスタンスを再作成する。2つ目はELBヘルスチェック。ELBからオートスケーリンググループの各インスタンスに対してヘルスチェックを行い、unhealthyなインスタンスを自動で削除して代替を再作成する。


##### CloudWatch Alarms ベースのオートスケーリングルール

CloudWatch alarms をもとにしてASGをスケールさせることができる
アラームはCPU使用率, ネットワーク使用率のようなメトリクスを監視する。
※メトリクスの計算はASG全体に対して平均して計算されるので注意すること。最大値・最小値ではなく、平均値。

アラームをもとにスケールインポリシー・スケールアウトポリシーをそれぞれ設定することができる

##### EC2メトリクスベースのオートスケーリングルール

EC2によって管理されるメトリクスを利用してオートスケーリングを設定することができる

- 平均CPU使用率
- ELBに存在するインスタンスあたりのリクエスト数
- 平均のNetwork in / Network out

##### カスタムメトリクスベースのオートスケーリング

接続中のユーザ数のようなカスタムメトリクスをベースにオートスケールを設定することができる

1. EC2上のアプリケーションからPureMetric APIを介してCloudWatchにカスタムメトリクスを送信する
2. カスタムメトリクスに対して、low / hight それぞれの値のCloudWatchアラームを設定する
3. 設定したCloudWatchアラームをASGのスケーリングポリシーに利用する



### EBS (Elastic Block Store)

ネットワークブロックデバイス.
AZをまたいだアタッチ・デタッチはできない。
簡単にアタッチ・デタッチができる他、スナップショットの作成も容易にできる。
ディスクサイズ変更もコンソールからできる。


### EFS (Elastic File System)

ネットワークファイルシステム(NFS).
AZをまたいでマウントできる。EBSより高いが、数百のインスタンスからマウントすることも可能。
