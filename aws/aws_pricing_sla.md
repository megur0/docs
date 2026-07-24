---
title: "料金・SLA・障害"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > 料金・SLA・障害


# AWSで発生している障害と影響の確認
* 「Personal Health Dashboard」で、リージョン単位の障害情報を確認できる。
* (例) EU-WEST-2リージョンで障害のステータスが上がっていたが、利用しているリージョンとは異なるため影響はなさそうと判断した(2020/8/25時点)。
* https://phd.aws.amazon.com/phd/home#/event-log

# AWSのSLA
* Amazonの利用規約では、EC2(EBS)はマルチAZ構成で99.99%(単体だと90%)。
    * 月換算: 24時間 × 30日 × 0.01% = 4.32分
* RDSのSLAはマルチAZで99.95%(シングルAZは保証対象外(?))。
    * 月換算: 24時間 × 30日 × 0.05% = 21.6分
* S3は99.9%、Lambdaは99.95%
* ELBはマルチAZ前提で99.99%
* KMSは99.9%
* (IMO) 上記を踏まえると、RDSよりAuroraを使うほうが良い(高可用性なマネージドDBサービスのため)。データベースを複数リージョンで整合性と速度を両立させて運用するのは難しく、マルチAZ構成自体が失敗するケースもあるため、Auroraを使うほうが確実だと思う。
* (参考) https://qiita.com/shojimotio/items/bf21180ec8436f58db43
* (参考) https://dev.classmethod.jp/articles/sla-for-more-than-100-aws-services/
