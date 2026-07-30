---
title: "ECS・ECR・EKS・Fargate - AWS"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > ECS・ECR・EKS・Fargate


## ECS(Amazon Elastic Container Service)
Amazon ECS(Elastic Container Service)は、クラスタ上でDockerコンテナの実行・停止・管理を行うフルマネージドのコンテナオーケストレーションサービス。

### タスク
* (参考) https://dev.classmethod.jp/articles/amazon-ecs-datamodel/
* Task: 1つ以上の、協調して動作するコンテナ群のまとまり。例えばWeb(Nginx)+AP(Spring Boot)のように互いに依存し合う構成は、1つのTaskとしてまとめて起動・停止する。
* 基本は1コンテナ=1関心事。疎結合な処理(別のTaskとして独立できるもの)は分離するのが一般的。

### サービス
* Service: Task Definitionをもとに起動したコンテナ群(Task)の数を維持し、ELB/ALBとの連携も担う概念。維持すべきTask数は`desired_count`で管理され、Taskが異常終了しても自動で補充される。AutoScalingでいうAutoScalingGroupに相当するもの。
* ローリングアップデートに対応しており、新Task起動→ヘルスチェック成功→旧Task停止の順に入れ替えるため、無停止でデプロイできる。
    ```
    旧Task 2台
        ↓
    新Task 2台起動
        ↓
    ヘルスチェック成功
        ↓
    旧Task停止
    ```

### ECSにおけるIAMロールの使い分け(重要)
ECSのタスクには役割の異なる2種類のIAMロールを設定できる。混同しやすいが目的が全く異なる。
* タスク実行ロール(Task Execution Role): ECSの基盤(コンテナエージェント)が使うロール。ECRからのイメージPull、CloudWatch Logsへのログ出力、Secrets Manager/Parameter Storeからのシークレット注入など、「コンテナを起動・運用するためにAWS側が裏で行う操作」の権限を持つ。AWS管理ポリシー`AmazonECSTaskExecutionRolePolicy`をアタッチするのが基本。
* タスクロール(Task Role): コンテナ内で動くアプリケーションコードが使うロール。「アプリがS3にファイルを置く」「SESでメールを送る」など、アプリケーション自身が行うAWS API呼び出しの権限を持つ。
* Web/API/Workerのようにコンテナの役割が複数ある場合、タスクロールをコンテナ(サービス)ごとに分けて必要最小限の権限だけを持たせるのが望ましい。例えばフロントエンド専用のコンテナがバックエンドAPI経由でしかAWSリソースを触らないなら、そのタスクロールにはAWS権限を一切付与しない、という設計もできる。
* タスク実行ロールは全サービス共通で1つにまとめてよいことが多い(コンテナ運用に必要な権限は基本的に共通のため)。

### クラスター
* Cluster: TaskはEC2(またはFargate)上で動作する。ClusterはそのTaskを稼働させる1つ以上のEC2の集合を指す。AutoScalingグループを想像しがちだが、単一のEC2や、AutoScalingしない複数EC2でもClusterとして成立する。
* EC2をClusterに参加させるには、Dockerコンテナとして動く`ecs-agent`をEC2上で起動しておく必要がある。ecs-agentがEC2の情報をECS側に送り、ECSはその情報をもとにTaskをどのインスタンスで起動するか決定する。
* AWSはDockerとecs-agentを事前設定済みのECS-Optimized AMIを提供しており、これを使うのが最も手軽。他のOS/ディストリビューションでも、ecs-agentさえインストールすればCluster Instanceとして利用できる。

### 用語整理
* タスク定義(Task Definition): コンテナの設計図(JSON)。あくまで設計図であり、これだけではコンテナは起動しない。

    | 項目 | 内容 |
    |------|------|
    | CPU / Memory | コンテナへ割り当てるリソース |
    | Container Image | 利用するDockerイメージ |
    | Port Mapping | コンテナが待ち受けるポート |
    | Environment | 環境変数 |
    | Secrets | Secrets Managerから取得する機密情報 |
    | IAM Role | AWSサービスへアクセスするための権限 |
    | Log Configuration | ログ出力先 |

* サービス(Service): タスク定義をもとにTaskを起動・維持する設定
* タスク(Task): サービスによって実際に起動されたコンテナ(群)
* (参考) https://qiita.com/VA_nakatsu/items/2e9235fd98b3c7eab507
* 3者の関係:
    ```
    Task Definition(設計図)
          ↓
    ECS Service(起動・維持)
          ↓
    Task(コンテナ)
          ↑
    Application Auto Scaling(Task数の自動調整)
    ```

コンテナサービスの種類として、それぞれ以下を選択可能。
* コントロールプレーン(=コンテナ管理をする場所): ECS、EKS
* データプレーン(=実際にコンテナが稼働する場所): EC2、Fargate
    * EKS+Fargateの組み合わせは2019年末にGAしており、現在は上記2×2の組み合わせすべてが選択可能。

### 環境変数とシークレットの注入
* コンテナ定義には、`environment`(平文)と`secrets`(Secrets Manager/Parameter StoreのARNを指定し、起動時にタスク実行ロールの権限で値を取得してコンテナに注入)の2種類の変数指定方法がある。
* パスワードやAPIキーなど機密性の高い値は`secrets`側に置き、コンテナ定義(タスク定義)自体には平文で残さないようにする。タスク定義はマネジメントコンソールやAPIから閲覧されうるため、機密値をそのまま`environment`に書くと閲覧できてしまう。

### デプロイ時の最小/最大ヘルシー割合
* サービス更新(ローリングデプロイ)時、既存タスク数に対してどこまで減らしてよいか(最小ヘルシー割合)、どこまで増やしてよいか(最大ヘルシー割合)を設定できる。
    * 例: 100%/200% → 既存タスクを維持したまま新タスクを同数追加起動し、ヘルスチェック成功後に旧タスクを止める(常に必要数以上を維持する、無停止デプロイ)。
    * 常時1台しか動かさない(スケールしない)サービスの場合、100%/200%のままだと新タスクの起動先リソースが足りず詰まることがあるため、最小ヘルシー割合を0%にして「一旦全部止めてから新タスクを起動する」構成にすることもある(その間は無停止にならない点に注意)。

### サービスのオートスケーリング
* サービスが維持すべきタスク数(必要数)をCPU使用率などのメトリクスに応じて自動増減させる仕組み(ターゲット追跡スケーリング)。目標値(例: CPU平均70%)を設定すると、その値に近づくようタスク数が自動調整される。
* IaCでインフラを管理している場合、オートスケーリングが書き換える必要数の値をIaC側の定義と衝突させない配慮が必要。そうしないと、IaCを適用するたびにオートスケーリングで増減させたタスク数が定義時の初期値に戻されてしまう(IaCツール側に「この値は管理対象外にする」設定が用意されていることが多い)。
* 常に固定台数で動かしたいワーカー(例: 多重起動できないバッチ/cron処理)は、そもそもオートスケーリングの対象外にし、タスク数を固定するのが安全。

### 構築の流れ
1. Dockerイメージをビルドして、ECRにpush
2. VPC、セキュリティグループ、サブネット、インスタンス用ロール(ECS)、キーペア、ELBを作成
3. ECSクラスターを作成(EC2起動タイプの場合、キャパシティ用にAuto Scaling Groupを別途用意し、Capacity Providerとして紐付けることが多い)
4. タスク用のロールを作成しておく
5. タスク定義(イメージ、コンテナ設定、ポートマップ)
6. サービス定義(タスクとの紐付け、ELBとの連携、Application Auto Scalingの設定など)
7. ELBのDNS名:`<port>`でブラウザからアクセスして確認

新しいイメージを作成した場合は、イメージをECRにpushし、タスク定義側で「新しいリビジョンの作成」を行ってイメージを差し替える。

### 異常系の挙動確認(検証メモ)
* EC2にログインしてコンテナを削除すると、自動でもう1つ作成される。
* ECSエージェントのコンテナを停止すると、自動で作成される。
* ECSインスタンス(EC2)を削除すると、1分ほどで起動し、サービスにも反映されコンテナも起動される。
* タスク起動時の失敗例: コンテナイメージ内のコードが誤っていてエラーになる、サービスでタスクを3つ起動しようとしたがインスタンスが2つしかなく各インスタンスタイプ(例: t2.micro)が1つのタスクしか持てない構成のため(合計2つしか起動できず)エラーになる、など。

### ログ
* デフォルトではコンテナ/インスタンスの再起動や停止でログが消えてしまうため、外部へ送る必要がある。いくつか方法があるが、CloudWatchに送るのが確実(IME)。
* `awslogs`ログドライバーを使うと、コンテナの標準出力(stdout/stderr)がそのままCloudWatch Logsへ送信される。アプリケーション側は`console.log()`などに出力するだけでよい。
    ```
    アプリケーション → 標準出力(stdout) → awslogs → CloudWatch Logs
    ```

### Fargateとの違い
* (参考) https://www.acrovision.jp/service/aws/?p=2599
* AWS FargateはAmazon ECSの起動時に選択できるデータプレーン(EC2かFargateかを選択)。Fargateの場合、サーバーやクラスターを一切管理することなくコンテナを実行できる。

### Fargateとlambdaの違い
* (IME) LambdaはFirebaseでいうCloud Functionsに近いイメージ。サーバー管理不要で、イベントをトリガーにプログラムを実行できる。
* FargateとLambdaの共通点は「インフラの構築・管理を削減し、アプリケーションの開発・運用に集中できる」点。料金体系はFargateがCPU/メモリの利用時間、Lambdaがリクエスト数と実行時間で決まるため、どちらが安いかは用途次第。
* (IME) 大規模なアプリケーションはFargate、小規模・イベント駆動な処理はLambdaを使うことが多い。Lambdaは実行時間などの制約が多く、大規模アプリではデプロイ時に問題が出やすいため、運用のしやすさの観点からFargate(状況に応じてEC2併用)を選ぶことが多い。

### EKS(Amazon Elastic Kubernetes Service)との違い
* Amazon EKSは、フルマネージド型のKubernetesサービス。セキュリティ・信頼性・スケーラビリティの要件が高いアプリケーションで選ばれることが多い(IME)。
* Kubernetesは、複数のDockerなどのコンテナを管理できるOSSのコンテナオーケストレーションツール。自動デプロイ、稼働中のスケーリング、自動修復(異常なコンテナの再起動・複製)などが特徴。
* (参考) https://kubernetes.io/case-studies/
* (参考) https://qiita.com/MahoTakara/items/85096f8b2632c802ab22
* (参考) [Amazon EKSによるスケーリング事例](https://employment.en-japan.com/engineerhub/entry/2019/12/12/103000)

### オートスケールについて(TODO)
* サービス・タスクのAuto Scale(Amazon ECSサービスの必要数を調整する仕組み)と、クラスターのオートスケール(通常のAuto Scale Groupと同じもの)の2つが存在する。
* サービス側のAuto Scaling(Application Auto Scaling)は、CPU使用率などのメトリクスを監視し、Serviceの`desired_count`を自動で書き換えることでTask数を増減させる仕組み。Taskを直接操作するのではなく、あくまでServiceが維持すべきTask数を変更しているだけ、という点がポイント。
    ```
    CPU使用率上昇 → desired_count: 2→4 → ServiceがTaskを4台まで起動
    ```
* 一方、クラスター側のオートスケールとどう連動するのか(EC2起動タイプの場合、Task数が増えてもインスタンスのキャパシティが足りなければ結局起動できないはずで、その連携がどう制御されているか)は理解しきれていない(TODO)。

### 参考リンク
* (参考) [動いているゲームでの活用例・構成](https://pages.awscloud.com/rs/112-TZM-766/images/I3-04.pdf)
* (参考) [デプロイフロー・トラブルシューティング](https://qiita.com/naomichi-y/items/d933867127f27524686a)
* (参考) [DockerFileのコマンドをAWSの設定で上書きする方法](https://qiita.com/akihiko_sugiyama/items/bc1628230e7ac58e09d1)

### FargateにSSHで接続(非推奨。参考情報)
* (IME) 非推奨の方法なのであくまで参考。SSM Agentで接続する方法がベター。
* SSHのpublic keyをParameter Storeに登録し、そのpublic keyをTask Definitionで読み込む設定をしてコンテナに渡す。シェルスクリプトでコンテナ内でそのpublic keyを`~/.ssh/authorized_keys`に登録し、同一VPC内のEC2からそのコンテナにSSHする(セキュリティグループ設定で、そのEC2からの22番ポートへのingressを許可しておく必要がある)。
* 同一VPC内のEC2はSSM Agentなどで接続する(FargateもSSM Agentで最初から接続できるのが望ましい(IMO))。
* (参考) https://qiita.com/masalennon/items/121c06c6f1d298a03d0c

### FargateでSSMエージェント
* (参考) https://qiita.com/ryurock/items/fa18b25b1b38c9a0f113
* (参考) https://enokawa.hatenablog.jp/entry/2019/09/05/104545
* (参考) https://qiita.com/pocari/items/3f3d77c80893f9f1e132
* (参考) https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ec2-run-command.html#run_command_iam_policy

## ENI(Elastic Network Interface)とFargate
* AWS Fargateを使うと、EC2インスタンスや仮想マシンを用意せずにコンテナを実行できる。コンテナは同一ホスト上でリソース・空間を共有して動くため可搬性は高いが、仮想マシンに比べて分離性は低く、セキュリティ対策は別途必要になる(追加料金で分離性を強化できる場合もある)。
* ENI(Elastic Network Interface): VPC内でIPアドレスなどのネットワーク情報を払い出す仮想ネットワークインターフェース。物理サーバーでNIC(Network Interface Card)を増設する代わりに、コンソール操作だけでネットワークインターフェースを追加できる。
* Fargateでコンテナを実行する場合、ネットワークモードは`awsvpc`が必須(EC2起動タイプのデフォルトは`bridge`)。`awsvpc`モードでは各TaskにENIとプライベートIPアドレスが付与されるため、Task単位でセキュリティグループを分けて通信制御できる。
    * (参考) https://www.acrovision.jp/service/aws/?p=2504
    * (参考) https://d1.awsstatic.com/webinars/jp/pdf/services/20190925_AWS-BlackBelt_AWSFargate.pdf
* Fargate内のコンテナ間通信は`127.0.0.1`宛てになる。そのため`docker-compose.yml`の`depends_on`/`links`を前提にサービス名で参照していた設定(`〜.conf`の`fastcgi_pass`など)は、そのままでは動かない点に注意。
    ```
    # 変更前(EC2の場合。Dockerのdepends_onやlinkで繋ぐイメージ)
    fastcgi_pass php-fpm:9000;
    # 変更後(Fargateの場合。depends_onやlinkがそのままでは機能しないため)
    fastcgi_pass 127.0.0.1:9000;
    ```
    * (参考) https://bunchon.hatenablog.com/entry/2019/04/08/231739
    * (参考) https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html

## ECR(EC2 Container Registry)
* Dockerのコンテナイメージを保存しておくためのレジストリ。ECRに保存したコンテナイメージは、Amazon ECSなどのサービスにデプロイできる。IAMを使った権限管理にも対応。コンテナイメージ自体はAmazon Simple Storage Service(Amazon S3)に保存される。
* 具体的な使い方
    1. ECRリポジトリを作成しておく
    2. AWS CLIで対象のプロファイルを選択しておく
    3. 以降のコマンドはAWSマネジメントコンソールでも確認できる
* (参考) https://qiita.com/3utama/items/b19e2239edb6996a735f

### ECRのイメージ管理
* リポジトリのタグ変更不可設定(イミュータブル)を有効にすると、同じタグへの上書きpushができなくなる。誤って同じタグ(例: `latest`)を使い回して意図しないイメージに差し替わる事故を防げる。
* push時のスキャンを有効にすると、pushされたイメージに対して自動で脆弱性スキャンが実行される。
* ライフサイクルポリシーで「保持するイメージ数」や「経過日数」を指定し、古いイメージを自動的に削除できる。無制限に貯め続けるとストレージ課金がかさむため、世代管理は基本的に設定しておく。
