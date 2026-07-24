---
title: "S3 - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > S3


# S3
* S3の特長は容量が無制限で安いこと。
    * S3の月額料金は0.025USD/1GBあたり(最初の50TB、スタンダードストレージ)。
    * EBSは0.12USD/1か月にプロビジョニングされたストレージ1GBあたり。
    * (IME) アプリケーションからアクセスする頻度が高いデータはEBSに入れておき、バックアップなどのデータをS3に置く構成がよく使われる。
* EC2<->S3の通信
    * (参考) https://avinton.com/academy/upload-file-from-ec2-to-s3/
    * EC2とS3の間でファイル転送を行いたい場合、S3はEC2が設置されているネットワーク(VPC)の外側にあるサービスであるため、エンドポイントと呼ばれるコンポーネントをVPCにアタッチしてS3と通信できるように設定する必要がある。
    * S3との通信はインターネットゲートウェイ経由でも可能だが、その場合トラフィックが一度インターネットへ出てしまう(同じAWSサービス同士の通信であるにもかかわらず)。
    * VPCエンドポイントを使えば、AWSのサービス内でセキュアにトラフィックを流すことができ、通信料金もかからないため経済的。
    * エンドポイントのポリシー作成時はデフォルトで全ての操作を許可するポリシーがセットされるため、S3以外の操作ができないように設定しておく。

## S3に静的ファイルを置いて、Route53からAレコードで飛ばす
* 概要: S3でバケットを作成し、静的ウェブサイトホスティングを設定する。
* (参考) [Amazon S3で静的ウェブサイトを設定する](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
* (参考) [Amazon S3バケットでホストされているウェブサイトへのトラフィックのルーティング](https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/RoutingToS3Bucket.html)

## S3の静的サイトをHTTPS化する
* AWSからSSL証明書を無料で発行できるサービス(Certificate Manager)がある。Certificate Managerで発行したSSL証明書はCloudFrontまたはElastic Load Balancingでのみ利用できる。EC2や他のVPSで運用しているサーバーには使えないので注意。
* (IME) サービス終了の告知用に静的ファイルだけをS3に置くようなケースでは、HTTPS証明書の設定をわざわざ行うのが手間になることがある。その場合はHTTPSアクセスをHTTPへリダイレクトさせる、という割り切った対応も一案。
* (参考) https://cre8cre8.com/aws/https-s3-and-cloudfront.htm
* (参考) https://dev.classmethod.jp/articles/cloudfront-s3-customdomain/

## Amazon S3 Glacier
* S3の料金の十分の一程度(?)。ただしS3自体が十分安いことと、GlacierはAPIツールが必要になる(?)ため、あまり使っていない(IME)。
* (参考) https://www.acrovision.jp/service/aws/?p=1646

## S3料金内容
* ストレージ容量、データ転送(課金対象はS3からの送信のみ。受信(S3へのアップロード)は無料)、リクエスト数で課金される。
    * (参考) https://qiita.com/kawaz/items/07d67a851fd49c1c183e
* CloudFront+S3とS3単体の料金比較: アクセスが多い場合はCloudFrontを使うほうがよい(?)。
    * (参考) https://qiita.com/yamamoto_y/items/c58ae2083a792d8b7b0f

## 使い方
* とりあえずバケットを作り、バケット自体の「ブロックパブリックアクセス」は全てオン状態にする。
    * この状態だと、個々のファイルにアクセスしても`access denied`になる。
    * PHPからSDKを使ってファイルをputする場合は、`'ACL' => 'private'`にしないと403 forbiddenになる(`public_read`などにするとエラーになる)。
    * バケット自体がパブリックアクセスをブロックしているため、個々のファイル側で許可を与えることはできない、ということ(?)。
    * 実際に個々のファイル(オブジェクト)の「アクセス権限」を見ると、「このバケットに対してブロックパブリックアクセス設定がオンになっているため、パブリックアクセスを許可することはできません。どの設定がオンになっているかを判断するには、ブロックパブリックアクセス設定を確認してください。」と表示される。
* アクセス権限について
    * バケット
        * ブロックパブリックアクセス: 全てオンにしておけば、とりあえずパブリックアクセスをブロックできる。
        * ACL: 他のAWSアカウントからのアクセス権限を付与できる。パブリックアクセスの設定もできる。
        * バケットポリシー: バケット単位の設定を管理する(?)。ブロックパブリックアクセスの「任意のパブリックバケットポリシーを介して、バケットとオブジェクトへのパブリックアクセスとクロスアカウントアクセスをブロックする」がオンになっていると機能しない。
    * 各オブジェクト
        * ACL: 個別にAWSアカウントからのアクセス、パブリックアクセスを設定できる。ただしブロックパブリックアクセスがオンになっていると、パブリックアクセスは設定できない。
    * (参考) https://dev.classmethod.jp/cloud/aws/s3-acl-wakewakame/
    * (参考) https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/acl-overview.html
* S3はCloudFrontを経由させ、Origin Access Identity(CloudFrontのみ許可)を使うことで閲覧制限ができる(?)。
