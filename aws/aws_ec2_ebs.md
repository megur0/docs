---
title: "EC2・EBS - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > EC2・EBS


# EC2
## インスタンスタイプの検索
* サイドメニューの「インスタンスタイプ」から検索できる。フィルタも使える。
    * 例: architectureはx86_64、金額(on-demand linux pricing)は0.1以下、メモリ8以上16未満、といった条件で絞り込める。この条件だと一番安いのは「t3a.large」で「0.0752 USD per Hour」だった(2021/3/15時点)。

## インスタンスタイプの変更
* EC2の場合は「停止」を実施してからインスタンスタイプを変更する。RDSでもインスタンスクラス変更時に再起動が発生するため、動き自体は近い(?)。

## インスタンスの削除
* インスタンスを停止する。数分待つとインスタンスの状態が「terminated」になる。数日してからリストから削除される。
* (参考) https://qiita.com/yuta-38/items/549087e77f1397bc1d92

## インスタンスを停止したのにEBSで課金される
* EBSストレージはアカウントでプロビジョニングしたストレージ量に応じて「ギガバイト-月」単位で課金される。
* EBS関連の課金を停止するには、不要になったEBSボリュームとEBSスナップショットを削除する。
* EBSスナップショットは、アクティブなEBSボリュームよりも安価な料金で請求される。
* 後からの使用に備えてEBSの情報は保持したままEBS課金を最小限にするには、バックアップ用にボリュームのスナップショットを作成してからアクティブなボリュームを削除する。後でスナップショットに格納していた情報が必要になった場合は、スナップショットからEBSボリュームを復元して利用できる。
* EBS: 1か月にプロビジョニングされたストレージ1GBあたり0.10USD
    * 1GBしか使っていなくても、100GBのプロビジョニングをしていると100GB分課金されるので注意。
* EBSスナップショット: 1か月に格納されたデータ1GBあたり0.05USD
* (参考) https://aws.amazon.com/jp/ebs/pricing/

## インスタンスを停止したのにElastic IPで課金される
* 次の条件が満たされている限り、Elastic IPアドレスに料金は発生しない。満たされていないElastic IPアドレスは、それぞれ1時間単位で請求される。
    * Elastic IPアドレスがEC2インスタンスに関連付けられている
    * Elastic IPアドレスに関連付けられているインスタンスが実行中である
    * インスタンスに1つのElastic IPアドレスしか添付されていない
* (参考) https://aws.amazon.com/jp/premiumsupport/knowledge-center/elastic-ip-charges/
* (参考) https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-billing-terminated/

## インスタンスを削除したのに課金される
* 例えば以下のEC2リソースがプロビジョニングされたまま残っている場合がある。
    * Amazon Elastic Block Store
    * Elastic IPアドレス
* 別のリージョンに実行中のインスタンスが残っている場合もある。
* Auto ScalingやElastic Beanstalkなどのサービスの設定によっては、インスタンスが自動的に再起動される場合がある。
    * 一部のサービスでは、削除されたインスタンスを置き換えるために自動的にインスタンスを起動することがある。これによりアプリケーションの耐障害性が高まる。代替インスタンスの起動を防ぐには、Auto Scalingグループを削除するか、Elastic Beanstalk環境を終了する。
    * (例) Auto Scalingグループを設定していると、メンテナンス中のインスタンスを置き換えるために新しいインスタンスが起動することがある。AWS Elastic Beanstalk環境には、通常デフォルトでAuto Scalingグループが含まれる。
* リザーブドインスタンスを購入している場合、契約が終了するまで毎月時間単位で請求される。
* (参考) https://aws.amazon.com/jp/premiumsupport/knowledge-center/terminate-relaunched-instance/

## CPUクレジットについて
* EC2のバーストパフォーマンスインスタンス(T2/T3系)も同様のCPUクレジットの仕組みで動作する。具体的な数値やRDSでの例は[RDS・Aurora](./aws_rds.md)の「CPUクレジットについて」を参照。

# EBS
* EC2のルートデバイスにEBSをアタッチする方法は以下を参照。
    * (参考) https://qiita.com/dskst/items/302119452bc399414bae
* デバイス名の例(?)
    * Amazon Linux(t2.micro): `/dev/xvda`
    * Ubuntu(t2.micro): `/dev/sda1`

## EBSの課金
* EC2とEBSは仮想サーバーとディスクの関係で、セットで使うのが基本。
* EC2を停止しても、紐づけられているEBSはプロビジョニングされている(保持している)だけで課金される(無料枠の期間中は課金されない)。
* 大量のデータを保存するなら、比較的安価なS3を使うほうがよい。
