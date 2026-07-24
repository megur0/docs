---
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > CloudWatch・CloudTrail


# CloudWatchとCloudTrail
* CloudWatch
    * AWSクラウドリソースとAWSで実行されるアプリケーションの監視サービス。CloudWatch/CloudWatch Logs/CloudWatch Eventsの3つのサービスで構成される。
* Amazon CloudTrail
    * AWSサービスの操作を監視し、AWSアカウントのガバナンス/コンプライアンス/運用とリスクの監査を行うマネージドサービス。ユーザーのAWSマネジメントコンソールへのログイン、AWSサービスに関する設定変更・APIを利用した操作、AWSサービスが実施した操作などのアクティビティログを記録する。標準で90日間、各リージョンでアクティビティを記録する。証跡を有効化すると、S3バケットにアクティビティログを保存できる。CloudWatch Logsへの送信を有効化すると、ログをCloudWatch Logsに送信できる。AWSコンソールまたはCloudTrail API/SDKで記録されているアクティビティを確認できる。
* CloudWatch Events: AWS上にあるリソースの状態変化をトリガーとしてアクションを実行する機能を提供する。
* CloudWatch Logs: AWSのログ監視サービスで、AWSマネージドサービスのログやEC2インスタンス上のOS/アプリケーションのログを取得する。1日から10年間の保持期間を指定するか、無制限にするか選択できる。
* CloudWatch: AWS上で稼働するAWSサービスのメトリクスを収集し、死活監視・性能監視・キャパシティ監視を行う。

# CloudWatchのアラーム・メトリクス
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
