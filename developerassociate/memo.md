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

#### tenancy

  default: 通常のインスタンス
  Dedicated instance: ハードウェア占有インスタンス
  Dedicated Host: ハードウェア占有ホスト
https://dev.classmethod.jp/articles/ec2-tenancy/ が参考になる


#### User Data

EC2の初回起動時の処理(インストール・設定など) をスクリプトで定義できる。これにより、初回起動でのセットアップを自動化できる
インスタンス作成時に指定可能

#### IP

- Public IP: 公開IP. sshやhttpで繋ぐ時などはこちらのIPをつかう
- Private IP: プライベートIP: AWS内のネットワーク上のIP
- Elastic IP: 固定IP. 所持している間課金される。任意のインスタンスにアタッチ・デタッチできる


#### Security Group

インスタンスやELBなどEC2のサービスにアタッチするinbound/outboundのルール
ip, CIDR, 他のセキュリティグループを元に通信の許可を定義できる
TCP, HTTPなどのプロトコルとポートを指定することができる

セキュリティグループは任意の数のインスタンスにアタッチでき、インスタンスは任意の数のセキュリティグループをもつことができる


