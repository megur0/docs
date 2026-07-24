---
title: "WAF・Shield - AWS"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > WAF・Shield


# AWS WAF
* AWS WAFを使うことで、以下のようなセキュリティ攻撃からWebサービスを防御できる。
    * SQLインジェクション攻撃
    * クロスサイトスクリプティング攻撃
    * OSコマンドインジェクション攻撃
    * DDoS攻撃
* AWS WAFは「API Gateway」「Elastic Load Balancer」「CloudFront」を使用している場合のみ攻撃から防御できる。
* AWS WAFはOSI参照モデルの7層(アプリケーション層)で行われるDDoS攻撃を緩和できるのに対し、AWS ShieldはOSI参照モデルの3層(ネットワーク層)および4層(トランスポート層)で行われるDDoS攻撃からWebサービスを防御する。

# AWS Shield
* リアルタイムにAWSへのトラフィックを検査し、悪意のあるものを検知する。
* OSI参照モデルの3層および4層で行われるDDoS攻撃からWebサービスを防御する。
* すべてのAWSユーザーが追加料金なしでDDoS攻撃への保護を受けられる「AWS Shield Standard」と、さらに高度なDDoS攻撃への備えとなる「AWS Shield Advanced」がある。Amazon EC2、Elastic IP、Elastic Load Balancing(ELB)、Amazon CloudFront、Amazon Route 53などのリソースで実行するアプリケーションを標的としたDDoS攻撃に対して、より高度な自動緩和機能を提供する。
