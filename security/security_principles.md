---
title: "セキュリティの基本原則・脆弱性管理 - セキュリティ"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(セキュリティ)](./README.md) > セキュリティの基本原則・脆弱性管理


# セキュリティの基本7原則
* Defense In Depth（多層防御）
* Least Privilege（最小権限の原則）
* System Hardening（システムの堅牢化）
* CIA Triad（セキュリティの3原則）
    * 機密性（Confidentiality）
    * 完全性（Integrity）
    * 可用性（Availability）
* Separation of Duties（職務分掌）
* AAAs of Security（認証・認可・監査）
    * Authentication（認証）: ユーザのアイデンティティ（本人性）を検証すること
    * Authorization（認可）: 認証に基づいて、権限を与えられたユーザのみがアクセスできるよう制御すること
    * Auditing（監査）: 認証・認可が適切に機能することを検証するため、認証・認可をログとして記録すること


# パッチ・マネジメント（Patch Vulnerable Systems and Software）
* システム・ソフトウェアは時間が経つにつれて様々な脆弱性が発見され、セキュリティレベルは低下していく傾向にある。適切にパッチを当てていくことでセキュリティレベルを維持する、という原則。
* 多くのインシデントは既知の脆弱性を悪用することで攻撃が成立するため、パッチ・マネジメントは重要。


# CVD（Coordinated Vulnerability Disclosure、協調的な脆弱性の公開）
* 発見された脆弱性に関する情報をベンダ（開発者）などの関係者と調整し、修正プログラムなどの解決策が用意できてから一般公開すること。
* https://en.wikipedia.org/wiki/Coordinated_vulnerability_disclosure


# CVE、CVSS
* CVSSスコアの目安
    * 7.0〜8.9: 高
    * 9.0〜10.0: 重大
* 本家
    * CVE
        * https://cve.mitre.org/
        * CVEは脆弱性の識別番号を採番するのが役割（IMO: スコア評価自体はCVE側では行っていない）
    * NVD（NISTのデータベース）
        * https://nvd.nist.gov/
    * (IMO) CVEが脆弱性を採番し、NVDなどがCVSSでスコア評価する、という役割分担になっていると理解している。
    * 最新の脆弱性をNVDで検索すると、スコアが「N/A（まだ評価されていない）」になっていることが多い。対象ソフトウェアの管理者サイトへのリンクが張られているので、最新の脆弱性内容や深刻度（CVSSスコアではなくCritical / Highなどの独自表記のことが多い）はそちらで確認できる。
* 日本
    * https://jvndb.jvn.jp/
    * (IME) 本家（CVE/NVD）より反映が遅くなる傾向がある。実際、OpenSSLの脆弱性が本家で公表された翌日昼になってもJVNにはまだ反映されていなかったことがあった。
