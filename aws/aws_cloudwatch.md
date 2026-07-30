---
title: "CloudWatch・CloudTrail - AWS"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > CloudWatch・CloudTrail


## CloudWatchとCloudTrail
* CloudWatch
    * AWSクラウドリソースとAWSで実行されるアプリケーションの監視サービス。CloudWatch/CloudWatch Logs/CloudWatch Eventsの3つのサービスで構成される。
* Amazon CloudTrail
    * AWSサービスの操作を監視し、AWSアカウントのガバナンス/コンプライアンス/運用とリスクの監査を行うマネージドサービス。ユーザーのAWSマネジメントコンソールへのログイン、AWSサービスに関する設定変更・APIを利用した操作、AWSサービスが実施した操作などのアクティビティログを記録する。標準で90日間、各リージョンでアクティビティを記録する。証跡を有効化すると、S3バケットにアクティビティログを保存できる。CloudWatch Logsへの送信を有効化すると、ログをCloudWatch Logsに送信できる。AWSコンソールまたはCloudTrail API/SDKで記録されているアクティビティを確認できる。
* CloudWatch Events: AWS上にあるリソースの状態変化をトリガーとしてアクションを実行する機能を提供する。
* CloudWatch Logs: AWSのログ監視サービスで、AWSマネージドサービスのログやEC2インスタンス上のOS/アプリケーションのログを取得する。1日から10年間の保持期間を指定するか、無制限にするか選択できる。
* CloudWatch: AWS上で稼働するAWSサービスのメトリクスを収集し、死活監視・性能監視・キャパシティ監視を行う。

## CloudWatchのアラーム・メトリクス
* アラーム
    * アラームをセットしておくと閾値を超えたときに通知を受け取れる。閾値を超えると「アラーム」状態になり、正常に戻ると「OK」になる。
* CPU
    * CloudWatchに表示されるCPU使用率はvCPU1個分の使用率。
* 見ておくべき項目
    * `CPUUtilization`
    * `DatabaseConnections`
    * `SwapUsage`: 使用可能なRAMの容量が減るとディスクから補うため増加する(?)。目安として50MBを超えないようにするのがよいとされる(?)。
        * (参考) https://docs.aws.amazon.com/ja_jp/AmazonElastiCache/latest/mem-ug/CacheMetrics.WhichShouldIMonitor.html#metrics-swap-usage
    * `FreeStorageSpace`: ディスク不足にならないよう確認する。
    * `FreeableMemory`: 使用可能なRAMの容量。
* (参考) https://hacknote.jp/archives/2069/

## Container Insights
* ECSクラスターで有効化すると、クラスター/サービス/タスク単位の詳細なCPU・メモリ使用率やログをCloudWatchで収集できるようになる(通常のECSメトリクスより粒度が細かい)。追加のログ収集・保存が発生するため、その分の追加料金がかかる点に注意。

## アラームの通知先と設計
* CloudWatchアラームの通知先には、SNSトピックを指定するのが基本形。SNSトピックにメールなどのサブスクライバーを登録しておけば、複数のアラームから同じ通知先へまとめて通知できる。
* アラームは「しきい値を超えた状態が何回連続したら発報するか」(評価期間の回数)を設定できる。瞬間的なスパイクで誤発報しないよう、ある程度の連続回数を要求するのが一般的。
* メトリクスが取得できない期間の扱い(欠測データの扱い)の設定を誤ると、監視対象が停止しているのに気づけない場合がある。「データが無い=異常とみなす」設定にしておくべき監視項目(常時稼働しているはずのサービスの起動数など)と、「データが無い=正常とみなす」でよい監視項目(オプショナルな処理のエラー数など)を区別する必要がある。
