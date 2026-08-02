---
title: "AWS CLI・CloudFormation - AWS"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > AWS CLI・CloudFormation


## CodeCommit
* Amazon Web Servicesのプライベートなgitリポジトリ。
* (参考) https://dev.classmethod.jp/articles/codecommit-introduction/
* (参考) https://dev.classmethod.jp/articles/codecommit-try/
* CodeCommit、CodeDeploy、CodePipelineを組み合わせることで、CI/CDを構築できる(?)。

## AWS CLI
### 利用ケース
* 自分のPCやAWS以外のサーバーから利用する場合
    * 専用のIAMユーザーを作成し、アクセスキーの発行が必要。
    * ただし2025年11月以降は、長期間有効なアクセスキーを発行せずに済む`aws login`コマンド([後述](#aws-login))という選択肢もある。
* EC2のインスタンス上から利用する場合
    * 必要な権限を持つIAMロールを作成し、EC2インスタンスにアタッチする(推奨)。
    * またはアクセスキーを利用する方法もあるが、アクセスキーが万が一流出した場合に外部から不正利用される可能性があるため非推奨。

### 新規AWSアカウントの初期セットアップ(IAMユーザー作成)
新しく作成したAWSアカウントでAWS CLIを使う場合、ルートユーザーの認証情報を直接使うのではなく、IAMユーザーを作成してアクセスキーを発行し、`aws configure`に設定する方法が一般的。

AWS CLIの認証方法は以下のいずれかを選べる。

|方法|利用可否|推奨度|
|-|-|-|
|ルートユーザーのアクセスキー|可能|非推奨|
|IAMユーザーのアクセスキー|可能|一般的|
|`aws login`(root/IAMユーザーの一時クレデンシャル、[後述](#aws-login))|可能(CLI 2.32.0以降)|個人開発では有力な選択肢|
|IAM Identity Center(SSO)|可能|推奨(組織・複数アカウント運用向け)|

ルートユーザーのアクセスキーでもCLIは利用できるが、ルートユーザーは全権限を持ち誤操作時の影響が最大になるうえ、アクセスキー漏洩時のリスクも大きいため非推奨。以下の構成が現実的。

```
ルートユーザー
  ↓
IAMユーザー作成
  ↓
アクセスキー発行
  ↓
aws configure
```

#### IAMユーザーの作成手順(コンソール)
```
IAM → Users → Create user
```
* User name: 用途が分かる名前にする(例: `terraform-admin`)。
* AWS Management Console access: CLI/Terraform専用に使うユーザーであればチェック不要。
* Permissions: `Attach policies directly`を選択し、`AdministratorAccess`をアタッチする。
    * Terraform等のIaCツールで多くのAWSサービス(VPC、ECS、RDS、ALB、IAM Role、S3、CloudFront、Route53、SESなど)を横断的に操作する場合、個別に権限を絞り込むよりもまずAdministratorAccessで構築し、必要に応じて後から権限を絞る方が現実的(IMO)。
* Tags: 任意(例: `Name: terraform-user`)。

#### アクセスキーの発行
```
IAM → Users → 対象ユーザー → Security credentials → Create access key
```
用途は`Command Line Interface (CLI)`を選択する。発行された「Access Key ID」と「Secret Access Key」は`aws configure`で使うため保存しておく。

### aws login
2025年11月に追加された、比較的新しいコマンド。IAM Identity Centerを使わずに、**既存のAWS Management Consoleのサインイン情報(ルートユーザー/IAMユーザー/フェデレーションID)をそのまま使って一時的なクレデンシャルを取得**できる。長期間有効なアクセスキーを発行・管理する必要がなくなる点が最大のメリット。

* **前提**: AWS CLI **2.32.0以降**が必要。
* IAMユーザーで使う場合は、そのIAMユーザー(またはロール・グループ)に管理ポリシー`SignInLocalDevelopmentAccess`をアタッチしておく必要がある。ルートユーザーで使う場合は追加の権限設定は不要。
* IAM Identity Centerを使っている場合は、この`aws login`ではなく`aws configure sso`/`aws sso login`([前述](./aws_organizations_identity_center.md))を使う。

```bash
aws login
```
* `--profile <name>`でプロファイルを指定/新規作成できる(未指定時は`default`)。
* ブラウザが自動的に開き、認証後にどの認証情報(ルート/IAMユーザー等)を使うか選択する。
* ブラウザの無い端末では`aws login --remote`を使うと、別デバイスで開くURLと認証コードが表示され、コードを手動で貼り付ける形で認証できる。

書き込まれる設定はアクセスキーではなく、対象IAMプリンシパルのARNのみ。
```ini
[default]
login_session = arn:aws:iam::111122223333:user/terraform-admin
region = ap-northeast-1
```

* 一時クレデンシャルは`~/.aws/login/cache`にキャッシュされる(SSOの`~/.aws/sso/cache`とは別ディレクトリ)。
* セッションは最大12時間有効で、15分ごとに自動リフレッシュされる。12時間を超えたら`aws login`をやり直す。
* ログアウトは`aws logout`(`--profile`指定可、`--all`で全プロファイル一括)。

IAM Identity Centerのような複数アカウント横断の一元管理はできないが、個人開発で「IAMユーザーは使いたいが長期アクセスキーは避けたい」という場合に、Identity Center/Organizationsを構築するほどではない規模でも使える手軽な選択肢(IMO)。

(出典: [Login for AWS local development using console credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html)、[Simplified developer access to AWS with 'aws login'](https://aws.amazon.com/blogs/security/simplified-developer-access-to-aws-with-aws-login/))

### （参考）AWS CLIでEC2インスタンスの作成
* VPC、IGWやセキュリティグループ、サブネット、SSH鍵の設定など、一通りAWS CLIから作成できる。
* (参考) https://www.isoroot.jp/blog/3188/

### ローカルでの利用
* (参考) https://qiita.com/shonansurvivors/items/1fb53a2d3b8dddab6629
* インストール(M1 Mac)
    * (参考) https://www.yamamanx.com/m1mac-aws-cliv2/
    ```
    curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    sudo installer -pkg AWSCLIV2.pkg -target /
    which aws
    aws --version
    ```
    * `brew install awscli`でインストールする方法もある。
    * (参考) https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-cmd-all-users

### 環境変数
* アクセスキーID: `AWS_ACCESS_KEY_ID`
* シークレットアクセスキー: `AWS_SECRET_ACCESS_KEY`
* リージョン: `AWS_DEFAULT_REGION`
* 出力形式: `AWS_DEFAULT_OUTPUT`
* プロファイル: `AWS_DEFAULT_PROFILE`

### プロファイルの設定(デフォルトとして設定される)
```
$ aws configure
  AWS Access Key ID [None]: {アクセスキー(各自)}
  AWS Secret Access Key [None]: {シークレットアクセスキー(各自)}
  Default region name [None]: (リージョンを指定)
  Default output format [None]: json
```
確認: `aws configure list`

* リージョン: 日本国内で使う場合は東京リージョンの`ap-northeast-1`を指定することが多い。
* 出力形式: `json`が無難(Terraformとの相性がよく、スクリプトでも処理しやすい)。他に`table`(人間には見やすい)、`text`(簡易表示)がある。

アカウントを取り違えていないかは以下でも確認できる。
```bash
aws sts get-caller-identity
```
```json
{
    "UserId": "AIDAxxxxxxxx",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-admin"
}
```
`Account`が意図したAWSアカウントIDになっていることを確認する。

### 名前付きプロファイル作成
```
$ aws configure --profile user1
```
すでに作成済みのプロファイルを指定すると編集モードになる。
確認: `aws configure list --profile user1`

### プロファイルの確認
* `~/.aws/credentials`、`~/.aws/config`に保存されている。専用の確認コマンドは無いため、以下で直接確認する。
```
cat ~/.aws/credentials
cat ~/.aws/config
```

### プロファイルの切り替え
```
export AWS_DEFAULT_PROFILE=user1
aws configure list  # 確認
```
* ターミナルを終了するとリセットされるため、デフォルトとして固定したい場合は`aws configure`で設定する。
* 元に戻す
```
export AWS_DEFAULT_PROFILE=default
aws configure list  # 確認
```
* `AWS_PROFILE`との関係(?): `AWS_DEFAULT_PROFILE`が設定されていない場合は`AWS_PROFILE`が優先されるらしい。両方設定されている場合は`AWS_DEFAULT_PROFILE`が優先される。

### 別アカウントへ切り替える(既存の設定を削除する場合)
プロファイルを使い分けるのではなく、古いAWSアカウントの設定自体を削除して新しいアカウントに一本化したい場合は、`~/.aws/credentials`・`~/.aws/config`を削除すればよい。

削除前に念のためバックアップを取っておく。
```bash
mkdir -p ~/.aws-backup
cp ~/.aws/credentials ~/.aws-backup/credentials.bak 2>/dev/null
cp ~/.aws/config ~/.aws-backup/config.bak 2>/dev/null
```

ファイル単位で削除:
```bash
rm -f ~/.aws/credentials
rm -f ~/.aws/config
```
またはAWS CLIの設定ディレクトリごと削除:
```bash
rm -rf ~/.aws
```

削除後に`aws configure`で新しいアカウントのIAMユーザーのアクセスキーを設定し直す。

### コマンド実行時にプロファイルを都度切り替える
* `--profile`で指定すればよい。(例) `aws s3 ls --profile user1`

### その他コマンドメモ
```
aws iam list-users   # IAMユーザー確認
aws iam list-groups  # IAMグループを確認
```

### EC2インスタンス上で使う場合
* (参考) https://qiita.com/toshiro3/items/37821bdcc50c8b6d06dc
* リージョン内で稼働している全インスタンスの情報を取得する場合
```
aws ec2 describe-instances
```

### 補足: 長期アクセスキーを避ける方法
アクセスキーを長期間発行し続ける運用はキー漏洩のリスクがあるため、避けられるなら避けたい(IMO)。選択肢は主に2つ。

* **[`aws login`](#aws-login)**: 既存のIAMユーザー/ルートユーザーのサインイン情報はそのまま使い、アクセスキーの代わりに一時クレデンシャルを使う。Identity Centerを構築するほどではない個人開発規模で手軽に導入できる。
* **IAM Identity Center(SSO)**: ブラウザ認証と一時的な認証情報でCLI/Terraformを利用できるうえ、複数AWSアカウントを1つのUserで横断管理できる。組織・複数アカウント運用ならこちらが本命(IMO)。

個人開発やTerraform学習用途で、複数アカウントを横断する予定が無いなら、IAMユーザー+アクセスキー方式のままでも実用上困らないが、`aws login`に切り替えるだけでアクセスキーを持たずに済むため、乗り換えの手間は小さい(IMO)。
* AWS Organizationsとの関係、Identity Centerのユーザー/Permission Set/アカウント割り当ての概念整理、`aws configure sso`〜`aws sso login`/`logout`の具体的な使い方は[Organizations・IAM Identity Center](./aws_organizations_identity_center.md)を参照。

## CloudFormation
* テンプレートファイルからAWSリソースをプロビジョニングするサービス。テンプレートベースで、作成〜変更〜削除が可能。CloudFormation自体への追加料金は無い。
* (参考) https://dev.classmethod.jp/articles/aws-all-iac/
* (参考) [入門レベルでわかりやすい記事](https://dev.classmethod.jp/articles/cloudformation-beginner01/)

### スタック
* テンプレートからプロビジョニングされるリソースの集合をスタックという。JSON/YAML形式のテンプレートテキストから、CloudFormationを経由してスタックを作成し、そのスタック単位でリソースの集合を管理する。
* スタックでリソースを管理しておくと、テンプレートを流した後に「設定を変えてやり直したい」となった際、スタックを削除するだけで簡単に作成したリソースを削除できる。

(例) YAML形式の場合(JSONやYAMLで書ける)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 〜〜
Parameters: # 実行時(スタック作成/更新)にユーザー入力を求めるパラメータ(KeyPairの名前や、DBのユーザー名など)
  ...

Resources: # Amazon EC2など、スタックを構成するリソースとプロパティ
  myVPC: # <Logical ID> テンプレート内で一意なID。テンプレート内で他のリソースを参照する際にこのIDを利用
    Type: AWS::EC2::VPC
    Properties: # 各リソースの作成時に指定するプロパティ。リソースタイプによって利用できるプロパティは異なるため、公式ドキュメントを確認しながら指定する
      CidrBlock: 10.0.0.0/16
      Tags:
      - Key: Name
        Value: first-VPC

Outputs: # スタック構築後にAWS CloudFormationから出力される値(DNSやEIPの値など)
  ...
```

`aws cloudformation`のような形でCLIから実行できる。WEBコンソールから実施すると再現性がイマイチになるため、CLIに統一するほうが無難(IMO)。以下のようなシェルファイルを用意しておくと便利。

```bash
#!/bin/bash

CFN_TEMPLATE=vpc.yml
CFN_STACK_NAME=vpc

aws cloudformation deploy --stack-name ${CFN_STACK_NAME} --template-file ${CFN_TEMPLATE} \
  --parameter-overrides \
  NameTagPrefix=prd \
  VPCCIDR=10.70
```

* (参考) 同じようなインフラのコード化として、OSSのTerraformもある。 https://qiita.com/jucky330/items/9868dca2b13366a6d074
* (参考) [組み込み関数リファレンス](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
* (参考) [テンプレートスニペット](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/CHAP_TemplateQuickRef.html)
* (参考) [応用編がかなり詳しい記事](https://dev.classmethod.jp/articles/aws-all-iac/)
