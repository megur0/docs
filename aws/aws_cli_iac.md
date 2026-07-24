---
title: "AWS CLI・CloudFormation - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > AWS CLI・CloudFormation


# CodeCommit
* Amazon Web Servicesのプライベートなgitリポジトリ。
* (参考) https://dev.classmethod.jp/articles/codecommit-introduction/
* (参考) https://dev.classmethod.jp/articles/codecommit-try/
* CodeCommit、CodeDeploy、CodePipelineを組み合わせることで、CI/CDを構築できる(?)。

# AWS CLI
## 利用ケース
* 自分のPCやAWS以外のサーバーから利用する場合
    * 専用のIAMユーザーを作成し、アクセスキーの発行が必要。
* EC2のインスタンス上から利用する場合
    * 必要な権限を持つIAMロールを作成し、EC2インスタンスにアタッチする(推奨)。
    * またはアクセスキーを利用する方法もあるが、アクセスキーが万が一流出した場合に外部から不正利用される可能性があるため非推奨。

## （参考）AWS CLIでEC2インスタンスの作成
* VPC、IGWやセキュリティグループ、サブネット、SSH鍵の設定など、一通りAWS CLIから作成できる。
* (参考) https://www.isoroot.jp/blog/3188/

## ローカルでの利用
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

## 環境変数
* アクセスキーID: `AWS_ACCESS_KEY_ID`
* シークレットアクセスキー: `AWS_SECRET_ACCESS_KEY`
* リージョン: `AWS_DEFAULT_REGION`
* 出力形式: `AWS_DEFAULT_OUTPUT`
* プロファイル: `AWS_DEFAULT_PROFILE`

## プロファイルの設定(デフォルトとして設定される)
```
$ aws configure
  AWS Access Key ID [None]: {アクセスキー(各自)}
  AWS Secret Access Key [None]: {シークレットアクセスキー(各自)}
  Default region name [None]: (リージョンを指定)
  Default output format [None]: json
```
確認: `aws configure list`

## 名前付きプロファイル作成
```
$ aws configure --profile user1
```
すでに作成済みのプロファイルを指定すると編集モードになる。
確認: `aws configure list --profile user1`

## プロファイルの確認
* `~/.aws/credentials`、`~/.aws/config`に保存されている。専用の確認コマンドは無いため、以下で直接確認する。
```
cat ~/.aws/credentials
cat ~/.aws/config
```

## プロファイルの切り替え
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

## コマンド実行時にプロファイルを都度切り替える
* `--profile`で指定すればよい。(例) `aws s3 ls --profile user1`

## その他コマンドメモ
```
aws iam list-users   # IAMユーザー確認
aws iam list-groups  # IAMグループを確認
```

## EC2インスタンス上で使う場合
* (参考) https://qiita.com/toshiro3/items/37821bdcc50c8b6d06dc
* リージョン内で稼働している全インスタンスの情報を取得する場合
```
aws ec2 describe-instances
```

# CloudFormation
* テンプレートファイルからAWSリソースをプロビジョニングするサービス。テンプレートベースで、作成〜変更〜削除が可能。CloudFormation自体への追加料金は無い。
* (参考) https://dev.classmethod.jp/articles/aws-all-iac/
* (参考) [入門レベルでわかりやすい記事](https://dev.classmethod.jp/articles/cloudformation-beginner01/)

## スタック
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
