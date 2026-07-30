---
title: "SSL証明書・ACM - AWS"
updated: 2026-07-30
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > SSL証明書・ACM


## AWSでのSSL証明書の取得方法
* 代理店などから購入する(有料)
* ACM(AWS Certificate Manager)を使う(無料、ただしEC2にはインストール不可。ELB、CloudFront、API Gateway、Elastic Beanstalkにはインストール可能)
* Let's Encryptを使う(無料、ただしAmazon Linuxは未対応(?))
* ACMでの証明書発行時は、ドメイン所有権の検証が必要になる。ホストゾーンがRoute53にある場合、検証用のCNAMEレコードをコンソールから自動でRoute53へ追加できる。

### CloudFront用ACM証明書はus-east-1固定
* ACMで発行した証明書は基本的に証明書を使うリソースと同じリージョンに作る必要があるが、CloudFrontだけは例外で、証明書を必ず`us-east-1`(バージニア北部)リージョンで発行する必要がある(CloudFrontはグローバルサービスのため)。
* そのため、同じドメインに対してALB用(利用リージョン)とCloudFront用(us-east-1)で証明書を2つ発行することになるケースがある。

#### (補足) AWSの自社認証局への移行(2021年頃の話)
* AWSのサービスは徐々にAWSの自社認証局から証明書を発行する方向へ移行が進んでいた。自社認証局の証明書に対応していないと、2021年3月以降にS3やCloudFrontの利用に影響が出る可能性がある、という案内があった。今となっては過去の移行対応の話であり、現在新たに対応が必要になる話ではない。
* (IME) ルート証明書の管理はアプリケーションによって異なるため対応方法は一概には言えないが、実際にはレアケースでほぼ影響がないことが多かった(?)。
* (参考) https://qiita.com/fukasawah/items/13f1657e1a055bc7d2c7
