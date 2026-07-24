---
title: "IAM・アクセス管理 - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > IAM・アクセス管理


# awsアカウント
* AWSアカウントとは、最初にサインアップしたときに作成されるアカウントのこと。すべてのリソース・サービスにアクセスできる強力なルート権限を持つ。
* そのため通常は使用せず、別途IAMユーザーを作成して運用することが推奨されている。

```
AWSアカウント(1個。ルート権限)
  └ IAMユーザー(N個)
```

* IAMポリシー
    * ポリシーは「AWSのどのリソースに対してどのような操作を許可するか」という「権限」を定義するもの。「誰が」その権限を使うかという情報は含まない。
    * ポリシーはユーザーやグループ、ロールにアタッチ(割り当て)して使う。
* IAMユーザー
    * AWSアカウントによって生成されるユーザー。複数作成できる。
    * IAMユーザーを作成せずにAWSアカウントを共有すると、フル権限のアカウントを皆で使うことになり危険。
    * 基本的な考え方として、AWSの操作を行う人間1人につきIAMユーザーを1つ作成する。
    * IAMユーザーはAWSアカウントに紐づく。作成直後は何も権限が与えられていない。
* IAMグループ
    * IAMユーザーの集合。1つのIAMユーザーを複数のグループに含めることも可能。
* IAMロール
    * IAMロールはユーザーやグループと似ているが、ユーザーやグループが「人」に対して割り当てられるのに対し、ロールは主に「(EC2等の)インスタンス」に割り当てられるケースが多い。
    * 例: EC2がS3との連携でファイルのアップロードや削除等の操作を自動で行うために、「MyEC2Role」というロールを作成し、それに「MyS3FullAccess」というポリシーをアタッチする。そのロールをEC2にアタッチすることで、EC2はS3に対する権限を得る。
    * 「人」が手動で行う操作の権限はユーザーやグループにポリシーをアタッチし、「インスタンス」が自動で行う操作の権限はロールにポリシーをアタッチしたうえで、そのロールをインスタンスにアタッチする。
    * ロールの割り当て先はインスタンスだけに限らない。AWSではフェデレーション等のSAML連携でログインすることも可能で、その場合SAMLユーザーのIDは別組織のIdP(IDプロバイダー)から提供される。そのSAMLユーザーが持つ権限も「ロール」のアタッチによって定める。
    * つまりロールの割り当てとは、IAMユーザーやIAMグループ以外の「何者か」に対して一時的に権限を与える仕組みだといえる。
    * 実際、ロールを割り当てられたエンティティは操作のたびにKMSの仕組みで「アクセスキー」と「シークレットキー」が発行され、その一時キーを使って操作を行う。
* ログイン
    * IAMユーザー
        * アカウントID(アカウント欄)、アカウント名、パスワードでログインする方法
        * または、専用URL(`ルートアカウント.signin.aws.amazon.com/console`)からアカウント名、パスワードでログインする方法
        * (TODO) この2種類のログイン方法が用意されている理由の使い分けは理解しきれていない。
    * ルートアカウント(通常は使用しない)
        * メールアドレス/パスワードでログイン
* AssumeRole
    * あるIAMロールを"一時的に"利用すること。具体的にはSTS(Security Token Service)が一時的なキーを発行することで、一時的にアクセス可能とする仕組み。このキーは数十分〜数時間で期限切れになる。
    * ※「一時的」に対する概念は永続的な利用で、これは例えばシークレットアクセスキーを発行してそのキーで永続的にアクセスすることを指す。キーが漏洩した場合のリスクが高い。
* AssumeRoleポリシー
    * そのロールを誰がAssumeRoleしてよいかを定めるポリシー。(例)「このロールはECSだけが利用できる」
    * ※ 一方、IAMポリシーは「AssumeRoleした後に何ができるか」を表す。

# IAM User/Group詳細
* IAM UserはUserName + Path、IAM GroupはGroupName + Pathから作成できる。
* Pathは、大規模なユーザー管理を行う場合にユーザーをディレクトリ分けするような仕組み。小規模な場合はデフォルトの`/`を使うことが多い。

# IAM Role詳細
* IAM Roleは以下3つの情報から作成できる。
    * RoleName
    * Path
    * AssumeRolePolicyDocument
* User、Group、Roleに対しては、作成後にPolicyを付与(put)できる。ユーザーやグループへのポリシー付与と同様、ロールに対しても同じことができる。違いは`AssumeRolePolicyDocument`という設定値が増えている点。
* (参考) https://dev.classmethod.jp/articles/iam-role-and-assumerole/#note-90289-8

## IAMロールの機能
* EC2インスタンスにAPIアクセス権限を委譲する機能(一般的によく知られている使い方)
* 自アカウントのAPI権限を委譲する機能(`sts:AssumeRole`)
* (参考) https://dev.classmethod.jp/articles/iam-role-and-assumerole/

## sts:AssumeRole
* AssumeRoleはSTS(Security Token Service)に対するAPIアクションの一つ。
* 「AssumeRoleはRoleArnを入力するとCredentialsを返すAPI」
    * RoleArn: IAM Roleの一意な名前で、`arn:aws:iam::123456789012:role/role-name`といった文字列。
* AssumeRoleの結果得られたAPIキーは、Roleに対して付与された権限を持つ。つまり、元々のユーザーが持っていた権限とは別の範囲の権限を、一時キーとして手に入れたことになる。
* ロールに設定された権限を持つ一時キーを入手することを「役割(role)の引き受け(assume)」と言う。自アカウントのロールを誰でも無邪気に引き受けられてしまうとセキュリティが保てないため、各ロールごとに、第三者に対して明示的に引き受けを許可する設定が必要。この設定が`AssumeRolePolicyDocument`。
* 「許可を受けた第三者」のことをTrusted Entityと呼ぶ。
* `AssumeRolePolicyDocument`は、ユーザーやグループに付与するPolicyと同じ形式で記述する。可変部は主にPrincipalの部分のみで、その他の部分は基本的に変更しない。
* 以下のように書くと、「EC2サービス」を信頼することにより「EC2がAssumeRoleを行えるように」なる。つまりEC2によるIAM Roleの利用は、「ロールによる権限委譲」という仕組みの上に構築された一機能だといえる。

```json
{
  "Version" : "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [ "ec2.amazonaws.com" ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

* 「ロールによる権限委譲」は以下のような応用もできる。
    * Facebookアプリを介してFacebookユーザーを信頼する(AWSをmBaaS的に使うイメージ)。GoogleやAmazonのIDでも実現できる。
    * Identity Providersとして独自のシステムをSAML Providerとして登録すれば、そのシステムに認証を委譲することも可能。

# ポリシー
* ポリシーは基本的に「誰が」「どのAWSサービスの」「どのリソースに対して」「どんな操作を」「許可する(許可しない)」かをJSON形式で記述する。
* 記述したポリシーをユーザー(IAMユーザー、IAMグループ、IAMロール)やAWSリソースに関連づけることで、アクセス制御を実現する。
* (参考) https://dev.classmethod.jp/articles/aws-iam-policy/

(例) AmazonS3ReadOnlyAccess: S3のリソースに対する参照操作を許可。
このポリシーをEC2インスタンスに関連づけると(正確にはIAMロールを経由して関連づける)、そのEC2インスタンス上のプログラム(AWS CLI、AWS SDKを利用したプログラム)からS3リソースに対する参照操作(Get、List)が可能になる。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:Get*",
        "s3:List*"
      ],
      "Resource": "*"
    }
  ]
}
```

## ポリシーの分類
* ユーザーベースのポリシー
    * このポリシーには「誰が」(操作する主体、ポリシーの記述項目で言うとPrincipal)の部分は記述しない。このポリシーを関連づけたIAMユーザー、IAMグループ、IAMロール(IAMロールをEC2インスタンスにアタッチした場合はEC2インスタンス)が「誰が」の部分に該当する。
    * 上の例では、AmazonS3ReadOnlyAccessポリシーを関連づけたIAMユーザー(やIAMグループ、IAMロール)がS3リソースを参照できる、ということ。
    * AWS管理ポリシー: AWSが用意している再利用可能なポリシー。複数のIAMユーザー、IAMグループ、IAMロール間で共有可能。
    * カスタマー管理ポリシー: ユーザーが作成した再利用可能なポリシー。複数のIAMユーザー、IAMグループ、IAMロール間で共有可能。
    * インラインポリシー: 各IAMユーザー(やIAMグループ、IAMロール)専用にユーザーが個別作成するポリシー。
* リソースベースのポリシー
    * 関連づける先がユーザーではなく「AWSサービス」であるポリシー。操作する主体(≒ユーザー)ではなく、操作される側(AWSリソース)に関連づける。よく使われるのはS3のバケットポリシー。記述内容のレベルでの違いは、操作主体を表すPrincipalの有無。
    * ユーザーベースのポリシーは「人(ユーザー、グループ、ロール)」に関連づけるため操作主体は暗黙的に決まるが、リソースベースのポリシーは「モノ(AWSリソース)」に関連づけるため暗黙的には決まらず、Principalの記載が必要。
    * (例) S3のバケットポリシー: 「AWSアカウント:777788889999のIAMユーザー:bob」が「example-bucket S3バケット配下」に「操作: PutObject、PutObjectAcl」を「許可する」

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::777788889999:user/bob"},
    "Action": [
      "s3:PutObject",
      "s3:PutObjectAcl"
    ],
    "Resource": "arn:aws:s3:::example-bucket/*"
  }
}
```

* リソースベースのポリシーはS3等の一部のAWSサービスのみに対応している。対応サービスは[IAMと連携するAWSサービスの表](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html)で「リソースベース」がYesになっている行を参照。Principalに指定できる値は[こちら](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_policies_elements.html#Principal)を参照。

## ポリシーでIAMユーザーの操作を制限する具体例(2022/1/24時点でのメモ)
* ポリシーで特定のResource、特定のActionに対してEffectをAllowにし、IAMユーザーにアタッチすることで、記載した操作以外を不可にできる。
* (参考) https://blog.ichikaway.com/entry/2017/11/28/174936
* (参考) https://yohei-a.hatenablog.jp/entry/20191014/1571009915
* Route53の特定のドメインしか編集できないようにする場合は以下のような設定になる。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "route53:ListTagsForResources",
                "route53:GetHostedZone",
                "route53:ChangeResourceRecordSets",
                "route53:ListResourceRecordSets",
                "route53:ListTagsForResource"
            ],
            "Resource": "arn:aws:route53:::hostedzone/ZXXXXXXXXXXXXX"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:GetHostedZoneCount",
                "route53:ListHostedZonesByName"
            ],
            "Resource": "*"
        }
    ]
}
```

* (参考) https://cloudpack.media/29549
* (参考) https://dev.classmethod.jp/articles/restrict-iam-policy-specific-instance-types/
* (参考) https://dev.classmethod.jp/articles/resource-permissions-for-ec2/

# セキュリティグループ
* (参考) https://ent.iij.ad.jp/articles/1486/

## 基本
* セキュリティグループは、EC2インスタンスの仮想ファイアウォールとして機能し、受信トラフィックと送信トラフィックを制御する。
* インバウンドルールはインスタンスへの受信トラフィックを制御する。
* アウトバウンドルールはインスタンスからの送信トラフィックを制御する。
* インスタンスの起動時に1つ以上のセキュリティグループを指定できる。
* セキュリティグループを指定しない場合、Amazon EC2はデフォルトのセキュリティグループを使用する(最大5つのセキュリティグループを割り当て可能)。
* トラフィックがインスタンスに到達することを許可するかどうかをAmazon EC2が判断する際、インスタンスに関連付けられているすべてのセキュリティグループのすべてのルールを評価する。

## ネットワークACL
* ネットワークアクセスコントロールリスト(ACL)は、1つ以上のサブネットのインバウンド/アウトバウンドトラフィックを制御するファイアウォールとして動作する、VPC用のセキュリティのオプションレイヤー。
* セキュリティの追加レイヤーをVPCに追加するには、セキュリティグループと同様のルールを指定したネットワークACLをセットアップできる。

## セキュリティグループとネットワークACLの違い
* (IME) AWSでよくある使い方として、ネットワークACLはデフォルト設定の「全て許可」のままにしておき、セキュリティグループでインスタンスへのアクセス制御を行うことが多い。
* セキュリティグループは「ステートフルなファイアウォール」で、インスタンスレベルで動作する。ステートフルなため、例えば外部からのSSHを許可する際はインバウンドにのみルールを記載すればよい。
* ネットワークACLは「ステートレスなファイアウォール」で、サブネットレベルで動作する。ステートレスなため、インバウンド・アウトバウンドそれぞれにルールを記載する必要がある。

## ソース
* セキュリティグループにはソース(送信元)を設定できる。送信元は、他のセキュリティグループ、IPv4/IPv6 CIDRブロック、単一のIPv4/IPv6アドレス、プレフィックスリストIDのいずれか。
* ソースにはセキュリティグループを指定することもできる。
    * (IME) セキュリティグループ(＋ネットワークインターフェース)自体が送信元のグルーピングのようにも機能するため、この仕組みは混同しやすく分かりづらいと感じる。
    * セキュリティグループをルールの送信元として指定すると、指定したセキュリティグループに関連付けられた「ネットワークインターフェース」からのトラフィックが許可される。
    * セキュリティグループに関連づけられたネットワークインターフェースを探すには、ネットワークインターフェース一覧で対象のセキュリティグループIDによりフィルタをかける。
    * (参考) https://ent.iij.ad.jp/articles/1486/

# ARN(Amazon Resource Name)
Amazon リソースネーム(ARN)は、AWSリソースを一意に識別する。IAMポリシー、Amazon RDSのタグ、APIコールなど、AWS全体に渡るリソースを指定する必要がある場合にARNが必要になる。

```
arn:partition:service:region:account-id:resource
arn:partition:service:region:account-id:resourcetype/resource
arn:partition:service:region:account-id:resourcetype:resource
```

* パーティション: 基本的に`aws`
* サービス: AWS製品名
* リージョン: リソースがあるリージョン。リージョンに関係ないものは省かれる
* リソース/リソースタイプ: サービスによって異なる。スラッシュやコロンなどで区切られる

EC2構文例:
```
arn:aws:ec2:region:account-id:instance/instance-id
arn:aws:ec2:region:account-id:volume/volume-id
```

S3構文例:
```
arn:aws:s3:::bucket_name
arn:aws:s3:::bucket_name/key_name
```
S3はリージョン・アカウントIDが不要。そのためバケット名がグローバルに一意である必要がある(?)。

RDS構文例:
```
arn:aws:rds:region:account-id:db:db-instance-name
arn:aws:rds:region:account-id:cluster:db-cluster-name
```
AWS CLI、RDS APIを使用する際に利用する。その際にはタグと共に使用する。
