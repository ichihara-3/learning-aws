# Leaning MEMO

## Udemy

### Ultimate AWS Certified Developer Associate 2020

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


- IAM policyはJSONで記述される

- IAM Federation
- 大きな企業では企業の認証情報を使ってログインすることができる

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

ターゲットグループの対象になるものはイアk

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



#### Network Load Balancer (v2)

L4向けロードバランサー。TCP, TLS, UDP をサポートする

数百万(millions)オーダーのリクエストを毎秒捌くことができる。
レイテンシは100ms未満程度(ALBは400ms程度)
NLBはAZごとに静的IPをもつ。また、ElasticIPを設定することができる。


##### NLBのユースケース

- 高パフォーマンスが要求される
- TCP / UDPレベルでのトラフィックを扱う必要がある


