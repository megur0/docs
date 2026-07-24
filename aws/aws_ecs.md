---
title: "ECS・ECR・EKS・Fargate - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > ECS・ECR・EKS・Fargate


# ECS(Amazon Elastic Container Service)
Amazon ECS(Elastic Container Service)は、クラスタ上でのDockerコンテナの実行・停止・管理をおこなえる、スケーラブルで高速なコンテナ管理サービス。

## タスク
* (参考) https://dev.classmethod.jp/articles/amazon-ecs-datamodel/
* 例えばWeb+APサーバーの構成で、WebサーバーにNginx、APサーバーにSpring Bootを採用した場合、サービスが2つ存在することになるため、それぞれを分離したコンテナとして起動すべきである。それぞれのコンテナは協調して動作することになり、どちらかが欠けていても動くものではなく、1つの塊として稼働するものになる。そういった1つ以上の協調して動作するコンテナ群を、ECSではTaskと呼ぶ。
* 一般的に、1コンテナにつき1サービスとして関心事を分離することが推奨されている。

## サービス
* Taskは、Task Definitionを元に実際に起動されたコンテナ群一式を指す。このコンテナ群を起動する数を調整したり、ELB/ALBとの連携を行ってくれるのがServiceという概念。Serviceは作成時に起動するタスクの数や紐付けるロードバランサーの指定を行う。Service内で起動すべきTaskの数はDesired Countという名前で管理されている。そのため、例えばTaskが何らかの理由で異常終了した場合、Serviceが新たにTaskを起動してTaskの数をDesired Countに保つ。AutoScalingで例えると、ServiceはAutoScalingGroupに該当するもの。

## クラスター
* コンテナを起動するにはOSが必要で、AWSにおけるOSとはEC2のこと。Clusterというのは、ECSのTaskを稼働させるための1つ以上のEC2の塊を指す。AutoScalingのEC2を想像しがちだが、AutoScalingではない複数のEC2インスタンスでも、単独のEC2インスタンスでもClusterとして成立する。
* EC2をClusterに参加させるには、ecs-agentがEC2上で起動されている必要がある。ecs-agentはDocker containerとしてEC2上で起動され、EC2の様々な情報をECS側に送る役割を持つ。ECS側ではecs-agentから送信される情報を基に、TaskをCluster上のどのEC2インスタンスで起動するか決定する。
* AWSはECS-Optimized AMIというAMIを提供している。Amazon Linux上にDockerやecs-agentといった、ECS Cluster用のEC2として稼働するための最小限の設定があらかじめ実施されているベースAMI。このAMIを利用するのが最も障壁が低いが、用途に応じてUbuntuなどのAmazon Linux以外のディストリビューションや、Container Linuxなどの軽量OS上にecs-agentをインストールしてCluster Instanceとして稼働させることも可能。

## 用語整理
* タスク定義(Task Definition): 1つ以上のコンテナ設定が記述されたJSONファイル
* サービス(Service): 定義されたタスクをECSクラスターで動作させるための設定
* タスク(Task): サービスによって実際に実行されたコンテナ(群)
* (参考) https://qiita.com/VA_nakatsu/items/2e9235fd98b3c7eab507

コンテナサービスの種類として、それぞれ以下を選択可能。
* コントロールプレーン(=コンテナ管理をする場所): ECS、EKS
* データプレーン(=実際にコンテナが稼働する場所): EC2、Fargate
    * EKSとFargateの組み合わせは長らく"coming soon"のままだったため(?)、選択可能な組み合わせは3通りだった(執筆時点)。

## 構築の流れ
1. Dockerイメージをビルドして、ECRにpush
2. VPC、セキュリティグループ、サブネット、インスタンス用ロール(ECS)、キーペア、ELBを作成
3. ECSクラスターを作成(Auto Scaling Groupも兼ねる(?))
4. タスク用のロールを作成しておく
5. タスク定義(イメージ、コンテナ設定、ポートマップ)
6. サービス定義(タスクとの紐付け、ELBとの連携など。ここに出てくるAuto Scaling設定はサービスのオートスケールを指す(?))
7. ELBのDNS名:`<port>`でブラウザからアクセスして確認

新しいイメージを作成した場合は、イメージを作成してECRにpushし、ECR側でタスク定義の「新しいリビジョンの作成」で「イメージ」を変更する。

## 異常系の挙動確認(検証メモ)
* EC2にログインしてコンテナを削除すると、自動でもう1つ作成される。
* ECSエージェントのコンテナを停止すると、自動で作成される。
* ECSインスタンス(EC2)を削除すると、1分ほどで起動し、サービスにも反映されコンテナも起動される。
* タスク起動時の失敗例: コンテナイメージ内のコードが誤っていてエラーになる、サービスでタスクを3つ起動しようとしたがインスタンスが2つしかなく各インスタンスタイプ(例: t2.micro)が1つのタスクしか持てない構成のため(合計2つしか起動できず)エラーになる、など。

## ログ
* デフォルトではコンテナ/インスタンスの再起動や停止でログが消えてしまうため、外部へ送る必要がある。いくつか方法があるが、CloudWatchに送るのが確実(IME)。

## Fargateとの違い
* (参考) https://www.acrovision.jp/service/aws/?p=2599
* AWS FargateはAmazon ECSの起動時に選択できるサービス(EC2かFargateかを選択)。Fargateの場合、サーバーやクラスターを一切管理することなくコンテナを実行できる。

## Fargateとlambdaの違い
* (IME) LambdaはFirebaseでいうCloud Functionsに近いイメージ。
* AWS Fargateとよく比較検討されるのがAWS Lambda。Lambdaを用いるとサーバーを使用せずにプログラムを実行できる。OSの構築やセキュリティ設定を行う必要はない。また、Lambdaの動作をLambda関数に登録することで、イベント発生時にそれをトリガーとしてプログラムを実行する、といった挙動も設定できる。
* FargateとLambdaの共通点は、過程は大きく異なるが「インフラの構築・管理を削減し、アプリケーションの開発・運用に集中できる」点。
* (IME) 大規模なアプリケーションを構築する場合はAWS Fargate、小規模の場合はAWS Lambdaを用いることが多いとされる。Fargateの料金は1時間に利用されたCPUとメモリの使用量、Lambdaの料金はリクエスト数とプログラムの実行時間で決まる。どちらがコストを削減できるかはシステムの用途によって変わるため一概には言えない。
* Lambdaは制約が多く、大規模アプリケーションでは特にデプロイ時に問題が発生することがあるため、運用・リリースをスムーズにする観点からFargateを用い、状況によってEC2と組み合わせることも多い(IME)。

## EKS(Amazon Elastic Kubernetes Service)との違い
* Amazon Elastic Kubernetes Service(Amazon EKS)は、フルマネージド型のKubernetesサービス。セキュリティ、信頼性、スケーラビリティを求める、機密性が高くミッションクリティカルなアプリケーションで利用されることが多い(?)。
* Kubernetes(クバネティス、クーベネティスと読む)は、複数のDockerなどのコンテナを管理できるオープンソースのプラットフォーム。
    * 自動デプロイ、稼働中のスケーリング(コンテナ数の変更)、新機能のシームレスな提供開始(ロールアウト)、ハードウェア利用率の最適化(コンテナの共存による稼働率向上)などを目的とする。
    * ゴールは、アプリの運用負担を軽減するエコシステムのコンポーネントとツールを整備すること。
        * 可搬性: パブリッククラウド、プライベートクラウド、ハイブリッドクラウド、マルチクラウド
        * 拡張可能: モジュール化、追加可能、接続可能、構成可能
        * 自動修復: 自動配置、自動再起動、自動複製、自動スケーリング
    * 2014年にプロジェクトが開始され、本番のワークロードを大規模に運用してきた経験とコミュニティのベストプラクティスを組み合わせて発展してきた。
    * (参考) https://kubernetes.io/case-studies/
    * (参考) https://qiita.com/MahoTakara/items/85096f8b2632c802ab22
* (参考) [Amazon EKSによるスケーリング事例](https://employment.en-japan.com/engineerhub/entry/2019/12/12/103000)

## オートスケールについて(TODO)
* サービス・タスクのAuto Scale(Amazon ECSサービスの必要数を調整する仕組み)と、クラスターのオートスケール(通常のAuto Scale Groupと同じもの)の2つが存在するが、これらがどう連動するのかを理解しきれていない(TODO)。

## 参考リンク
* (参考) [動いているゲームでの活用例・構成](https://pages.awscloud.com/rs/112-TZM-766/images/I3-04.pdf)
* (参考) [デプロイフロー・トラブルシューティング](https://qiita.com/naomichi-y/items/d933867127f27524686a)
* (参考) [DockerFileのコマンドをAWSの設定で上書きする方法](https://qiita.com/akihiko_sugiyama/items/bc1628230e7ac58e09d1)

## FargateにSSHで接続(非推奨。参考情報)
* (IME) 非推奨の方法なのであくまで参考。SSM Agentで接続する方法がベター。
* SSHのpublic keyをParameter Storeに登録し、そのpublic keyをTask Definitionで読み込む設定をしてコンテナに渡す。シェルスクリプトでコンテナ内でそのpublic keyを`~/.ssh/authorized_keys`に登録し、同一VPC内のEC2からそのコンテナにSSHする(セキュリティグループ設定で、そのEC2からの22番ポートへのingressを許可しておく必要がある)。
* 同一VPC内のEC2はSSM Agentなどで接続する(FargateもSSM Agentで最初から接続できるのが望ましい(IMO))。
* (参考) https://qiita.com/masalennon/items/121c06c6f1d298a03d0c

## FargateでSSMエージェント
* (参考) https://qiita.com/ryurock/items/fa18b25b1b38c9a0f113
* (参考) https://enokawa.hatenablog.jp/entry/2019/09/05/104545
* (参考) https://qiita.com/pocari/items/3f3d77c80893f9f1e132
* (参考) https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ec2-run-command.html#run_command_iam_policy

# ENI(Elastic Network Interface)とFargate
* AWS EC2はEC2インスタンスを用いて仮想マシンを用意し、そのインスタンス上にサーバーを構築する必要があったのに対し、AWS FargateではEC2インスタンスや仮想マシンを用意する必要がなく、コンテナを用いることでメモリ使用率を抑えつつアプリケーションサーバーの構築を迅速に行える。
* 仮想マシンは、ホストとなるマシンの上に複数台の仮想OSを立て、それぞれのユーザーが使用できるOSを提供するのに対し、コンテナは同じホストマシンの上にそれぞれのユーザーが使用できるリソースと空間を提供する。コンテナはコンテナエンジンをベースに動作するため、コンテナエンジンさえあればどんな状況でも動作させられ、可搬性が高いというメリットがある。
* 一方でコンテナにはデメリットもある。1つのホストマシン上で動作するため、仮想マシンのように個別に異なるOSを選ぶことはできない。OSを構築する手間を省いているぶんメモリ使用率の節約や構築時間の低減に成功しているが、複数種類のOSを開発で利用する必要がある場合は無駄が多くコストがかさむ可能性がある。
* また、同じマシン上で動作する関係上、仮想マシンと比べてコンテナ同士の分離性が低い。分離性が低いと、セキュリティレベルが低い場合に1つのコンテナが外部から攻撃を受けた際、他のコンテナも脅威にさらされる可能性がある。そのためセキュリティ面でも十分な対策が必要になる(追加料金を払うことで分離性の強化にある程度対応できる場合もある)。
* コンテナの管理ツールであるAWS ECSをシステムの基盤として利用しており、AWS EC2の場合はインスタンスの利用数やそれぞれの管理、クラスターの管理などが必要だったのに対し、AWS Fargateではそういったインスタンス管理は不要で、コンテナイメージを構築しCPUやメモリといったリソースを指定するだけですぐに利用できる。CPUなどのリソースのスケーリングも自動で行われ最適化される。
* AWS ENIはElastic Network Interfaceの略称。AWS VPC(Amazon Web Services Virtual Private Cloud)と呼ばれる、ユーザーごとに個別に提供されるプライベートなクラウドにおいて、IPアドレスなどのネットワーク情報をENIに付与することで、より柔軟なネットワーク構成を実現する仮想ネットワークインターフェース。
* 物理的な環境でサーバーの役割に応じて複数のIPアドレスを持たせるなど、ネットワークインターフェースを増やすにはサーバーにNIC(Network Interface Card)を追加で挿す必要があるが、AWS ENIを用いればクラウド上でIPアドレスの登録やセキュリティグループの登録を行うだけで簡単にネットワークインターフェースを追加できる。
* Fargateでコンテナを用いてECSタスクを実行する場合、ECSはネットワークとしてawsvpcネットワークモードを必要とする。このネットワークモードでは各タスクにENIとプライマリプライベートIPアドレスが付与される。awsvpcネットワークが各タスクにENIを付与するため、タスクの種別ごとにセキュリティグループを分けて通信制御を行える、というメリットがある。awsvpcネットワークはAWS Fargate固有の機能というわけではないが、AWS Fargateのネットワークの裏側ではENIが用いられている。
* (参考) https://www.acrovision.jp/service/aws/?p=2504
* Fargateはネットワークモードがawsvpcモードのみ(EC2などはデフォルトがbridgeモード)。
    * Fargateだと、awsvpcネットワークモードで、VPC内の他のリソースへプライベートIPで通信が可能になる(?)。
    * (参考) https://d1.awsstatic.com/webinars/jp/pdf/services/20190925_AWS-BlackBelt_AWSFargate.pdf
* Fargate内のコンテナ間通信は`127.0.0.1`での通信になる。そのため、`docker-compose.yml`などで`depends_on`や`links`にして、`〜.conf`ファイルなどでサービス名を参照している部分はそのままだと動かないので注意。
    * 例:
    ```
    # 変更前(EC2の場合。Dockerのdepends_onやlinkで繋ぐイメージ)
    fastcgi_pass php-fpm:9000;
    # 変更後(Fargateの場合。depends_onやlinkがそのままでは機能しないため)
    fastcgi_pass 127.0.0.1:9000;
    ```
    * (参考) https://bunchon.hatenablog.com/entry/2019/04/08/231739
    * (参考) https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html

# ECR(EC2 Container Registry)
* Dockerのコンテナイメージを保存しておくためのレジストリ。ECRに保存したコンテナイメージは、Amazon ECSなどのサービスにデプロイできる。IAMを使った権限管理にも対応。コンテナイメージ自体はAmazon Simple Storage Service(Amazon S3)に保存される。
* 具体的な使い方
    1. ECRリポジトリを作成しておく
    2. AWS CLIで対象のプロファイルを選択しておく
    3. 以降のコマンドはAWSマネジメントコンソールでも確認できる
* (参考) https://qiita.com/3utama/items/b19e2239edb6996a735f
