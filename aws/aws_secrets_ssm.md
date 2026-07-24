---
title: "Secrets Manager・SSM(Systems Manager)"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > Secrets Manager・SSM(Systems Manager)


# シークレットマネージャ
## 秘密鍵をバイナリデータとして登録する
* 概要
    * テキストデータとして登録する場合の問題点
        * AWSの画面からそのまま秘密鍵を登録すると、改行コードがスペースに置き換えられてしまう。
        * 取り出した後に改行コードを変換すれば対処できなくはないが、そもそも画面に生の秘密鍵が表示されるのは望ましくない。
    * シークレットマネージャはテキストデータだけでなくバイナリデータも登録できるため、それを利用する。
        * バイナリデータとして秘密鍵を扱えば、生のまま扱いやすくなる。
        * バイナリデータの登録・参照はコンソールではサポートされていないため、CLIを使う。
        * バイナリデータを登録する際はBase64でエンコードする必要があるが、登録時はAWS CLIが自動でエンコードしてくれる。
        * 秘密鍵自体がbase64で元々エンコードされている場合、それをさらにbase64にエンコードするのは冗長に感じるが、バイナリデータとしてシークレットマネージャに登録するための仕様なので仕方ない(?)。
* 格納
```
aws secretsmanager create-secret --name sample-github-secret --secret-binary fileb://~/.ssh/github/id_rsa
```
```json
{
   "ARN": "arn:aws:secretsmanager:us-east-1:123456789012:secret:sample-github-secret-klASQT",
   "Name": "sample-github-secret",
   "VersionId": "d6ea7210-b20f-44d3-8950-10566842dce7"
}
```
* `fileb://`というプレフィックスをつけることで「ファイルの内容をエンコードされていないバイナリとして扱う」ことになる。
* AWS CLIバージョン2の場合はデフォルトでバイナリパラメータをbase64エンコードされた文字列として渡すため、`file://`にするとbase64エンコードされずエラー(invalidなbase64というエラー)になる。
* 取得
    * `aws secretsmanager get-secret-value --secret-id ${KEYPAIR_NAME}`でbase64エンコードされた文字列を確認できる。
    * 実際に取り出す際のサンプル
    ```
    sudo aws secretsmanager get-secret-value --secret-id sample-github-secret --region us-east-1 --query 'SecretBinary' --output text | base64 -d > /home/ec2-user/.ssh/id_rsa
    ```
* (参考) https://dev.classmethod.jp/articles/managing-keypairs-with-secrets-manager/
* (参考) https://dev.classmethod.jp/articles/manage-binary-secrets-with-aws-secrets-manager/
* (参考) https://dev.classmethod.jp/articles/got-into-about-blob-type-arguments/

# SSM(Systems Manager)/SSM Agent
* EC2インスタンスを立てて、Amazon Linux 2などであればデフォルトでSSM Agentが入っている(?)。
* AWSコンソール画面で、EC2のロールに`AmazonEC2RoleforSSM`ポリシーを割り当てると接続できるようになる。ターミナルからもコンソール画面からも接続可能(ただし、割り当てから接続できるようになるまでに多少のタイムラグがある。すぐに接続しようとすると「SSMエージェントがインストールされていない」といったエラーが出ることがある)。
* 接続すると`whoami`で`ssm-user`と表示される。
* (参考) https://fu3ak1.hatenablog.com/entry/2020/05/30/141650
* (参考) https://qiita.com/mksamba/items/6d7a0b84894578feafa8

## 設定
* EC2がSystems Managerの機能を使用するために、EC2にSystems Managerの利用権限が必要。EC2のIAMロールに`AmazonEC2RoleforSSM`ポリシーを付与する必要がある。
* 対象のEC2からSystems Managerのエンドポイントへアクセスできるようにネットワーク設定をしておく必要がある。例として以下のパターンがある。
    * publicなsubnet+EIP付与パターン: IGW経由でエンドポイントへアクセス(EIPが無いと接続できない)。
    * privateなsubnet+NAT gatewayパターン: EC2 -> NAT gateway -> IGW経由でエンドポイントへアクセス(private subnet上のルートテーブルにNAT gateway(public subnet)への経路を設定しておく必要がある)。
    * privateなsubnet(インターネットとの接続なし): 「VPCエンドポイント」経由でエンドポイントへアクセス。
        * `com.amazonaws.ap-northeast-1.ssm`
        * `com.amazonaws.ap-northeast-1.ec2messages`
        * `com.amazonaws.ap-northeast-1.ssmmessages`
* SSM AgentがデフォルトでインストールされていないEC2を使用する場合は、「ユーザーデータ」(EC2の高度な詳細で設定できる)を使ってインストールする必要がある。Amazon LinuxやAmazon Linux 2を使う場合はデフォルトでインストールされているため、この設定は不要。

## 確認
* 「AWS Systems Manager」-「インスタンスとノード」-「マネージドインスタンス」のメニューを開き、対象のインスタンスが「マネージドインスタンス」として表示されていることを確認する。
* 「アクション」-「Start Session」を選択すると、ブラウザから接続できる。SSHで接続するのと同様にコマンドを実行できる。

## AWS CLIから接続
* Session Manager pluginをインストールしておく。
    * (参考) https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html#install-plugin-macos
* `--target`オプションに接続先インスタンスのインスタンスIDを指定するだけで接続できる。
```
aws ssm start-session --target i-0c45bdd6943767bd0
```
* インスタンスIDの確認
```
aws ssm describe-instance-information
```

## ログ
* 接続の概要(接続したIAMユーザー、接続先インスタンス、時刻)はマネジメントコンソールで確認できる。
* 接続してどんなコマンドを打ったかの詳細をS3バケットやCloudWatch Logsに出力することも可能。そのためには以下の追加設定(3点)が必要。
    * Roleへの権限追加: インスタンスに付与しているRoleに、S3バケットおよびCloudWatch Logsへのアクセス権を追加する(検証時は`S3FullAccess`、`CloudWatchLogsFullAccess`を追加したが、本来は必要最小限にすべき)。
    * S3エンドポイントの追加: インターネット接続が無い環境ではS3エンドポイントの追加が必要。VPCエンドポイント`com.amazonaws.ap-northeast-1.s3`を追加する。
    * セッションマネージャーでのログ出力の設定: 「AWS Systems Manager」-「インスタンスとノード」-「セッションマネージャー」-「設定」で、セッションのログをS3バケットおよびCloudWatch Logsに出力する設定を追加する(S3はバケット、CloudWatch Logsはロググループを指定)。
* SSMエージェントログ
    * 通常は`/var/log/amazon/ssm/amazon-ssm-agent.log`、`/var/log/amazon/ssm/errors.log`に出力される。
    * `/etc/amazon/ssm/`の`seelog.xml.template`が設定テンプレートなので、これをコピーして`seelog.xml`を設定・配置する。
    * (参考) https://dev.classmethod.jp/articles/change-ssm-agent-log-configuration/

## 注意点
* SSMで接続すると`/var/log/ssm/`配下にSSM接続時の操作ログが残る。このログでディスク容量が圧迫される可能性もあるため、ログサイズとローテーションの設定を必ずしておく。
* デフォルトでは`ssm-user`という権限の強いユーザーで接続する(パスワード無しでroot権限に昇格できる)。本番環境で利用する際はRun-Asの設定で接続するOSユーザーを指定したり、実行できるコマンドを制限することを検討する。
* OSのパスワード入力は不要で、IAMの権限のみでサーバーに接続できる。そのためIAM権限をむやみに渡さないなど、慎重に扱う必要がある。
* 多くのコマンドを実行する場合は、マネジメントコンソール(ブラウザ)よりAWS CLIを使ったSession Manager接続がおすすめ。
* (TODO) `ssm-user`でディレクトリを作成しても所有者はrootになる、`script`コマンドを実行するとrootに勝手に切り替わる、など、内部的な仕組みを理解しきれていない部分がある。

## FargateでSSMエージェント
* (参考) https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ec2-run-command.html#run_command_iam_policy

# Parameter StoreとSecrets Managerの使い分け
* (参考) https://aws.amazon.com/jp/systems-manager/faq/
* Q: パラメータストアとシークレットマネージャーのどちらを使えば良いか?
    * A: 設定とシークレットにひとつのストアが欲しい場合はパラメータストアを、ライフサイクル管理を備えたシークレット専用のストアが欲しい場合はシークレットマネージャーを使う。パラメータストアは追加料金なしでパラメータ数10,000個までの制限で使える。
* SSM(Systems Manager)のパラメータストアでは、以下3種類をサポートしている。
    * String(平文)
    * String List(平文)
    * Secure String(暗号化)

## CDKでのパラメータストア
* パラメータを安全に扱いたい場合は、インターネット上に流れないようにする。そのため、テンプレート作成時ではなくデプロイ時に動作する関数を使う。
* (参考) https://dev.classmethod.jp/articles/aws-cdk-ssm-secrets-manager/

## Secure Stringの取得
* Secure String(暗号化)の取得は特定の用途でしか使えないため(?)、実際にシークレットを管理したい場合はAWS CLIから取得するほうがよさそう(IME)。
```
aws ssm get-parameter --name (parameter-name) --with-decryption --output text
```
* `--name`: パラメータ名を指定
* `--with-decryption`: パラメータの復号を行う
* `--output`: 出力形式
* `--query Parameter.Value`を加えることで、パラメータ値のみ取得できる。
* (参考) https://qiita.com/tyoshitake/items/d62ff2ebce9482d84096
* もしくはシークレットマネージャを使う方法もある。
    * (参考) https://mano.hatenadiary.jp/entry/cdk-bastion

## パラメータストア格納
* パラメータがインターネット上に流れるのは望ましくないため、インターネット経由の操作はなるべく避ける。
```
aws ssm put-parameter --name "ops-activation-code" \
  --value "xxxxxxxxxxxxxxxxxxx" \
  --type "SecureString"
```

## CDKからパラメータを取り出す方法
* `valueFromLookup`
* `valueForStringParameter`
* `valueForSecureStringParameter`
* (参考) https://dev.classmethod.jp/articles/aws-cdk-ssm-secrets-manager/

## SSH鍵をパラメータストアに登録する
* (参考) https://qiita.com/tyoshitake/items/d62ff2ebce9482d84096
