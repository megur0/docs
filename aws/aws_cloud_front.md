---
title: "CloudFrontの構成パターン - AWS"
updated: 2026-07-30
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > CloudFrontの構成パターン


## CloudFront

### CloudFrontとは
CloudFrontはAWSのCDN（Content Delivery Network）であり、ユーザーからのリクエストを受けて適切な配信元（Origin）へ転送し、必要に応じてコンテンツをキャッシュするサービス。
* 静的ファイルおよび動的コンテンツを、ユーザーに最も近いエッジサーバーから配信できる。配信サーバーは自動的に選択されるため、利用者側で特別な設定は不要。
* 動画や音声のメディアストリーミングデータの配信にも対応。
* 使い道: サーバーの負荷軽減、静的・動的コンテンツのキャッシュなど。

### 基本用語

#### Origin
CloudFrontがコンテンツを取得する配信元。
例：ALB、S3、EC2、API Gateway

1つのDistributionに複数のOriginを登録できる。

#### Behavior
どのパスをどのOriginへ転送するかを決めるルール。

例：
- `/api/*` → ALB
- `/cdn/*` → S3
- その他 → ALB

CloudFrontはURLのパスを見て転送先を判断する。

### キャッシュ
CloudFrontは「画像だからキャッシュする」のではなく、Behaviorに設定されたCache Policyによってキャッシュするかどうかを決める。

代表的なManaged Policy
- **CachingDisabled**：キャッシュしない（API向け）
- **CachingOptimized**：積極的にキャッシュする（画像・動画・CSS・JSなど向け）

**OriginがALBでもS3でもキャッシュ可能**。CloudFrontはOriginの種類ではなく、BehaviorとCache Policyによってキャッシュを制御する。

#### Cache PolicyとOrigin Request Policyの違い
- **Cache Policy**：「何をキャッシュキーに含めるか」「TTLをどうするか」を決める。同じキャッシュキーのリクエストはOriginへ転送されずCloudFrontから直接返される。
- **Origin Request Policy**：キャッシュの有無に関わらず、Originへリクエストを転送する際に「どのヘッダー/Cookie/クエリ文字列を転送するか」を決める。

例えばAPIのようにキャッシュ自体はしない（CachingDisabled）が、認証ヘッダーやCookieはOriginへそのまま転送したい、というケースではOrigin Request Policy側で転送対象を指定する。AWSは両方について代表的な設定をマネージドポリシーとして用意しており、自前で作らずそれらを選ぶだけで利用できることが多い。

### OAC（Origin Access Control）
CloudFrontからのみS3へアクセスできるようにする仕組み。

```text
Browser
    ↓
CloudFront
    ↓
S3
```

S3への直接アクセスを防ぎ、安全にコンテンツを配信できる。

S3側のバケットポリシーでOACを許可する際は、許可対象を特定のCloudFront Distributionに限定する条件（そのDistributionのARNと一致するかどうか）を付けておくと安全。単にCloudFrontのサービスプリンシパルを許可するだけだと、同じアカウント内の別のDistributionからも誤ってアクセスできてしまう。

### CloudFrontの代表的な構成

#### ① 静的サイト配信

```text
Browser
    ↓
CloudFront
    ↓
S3
```

HTML・CSS・JS・画像など、すべてS3から配信する。

#### ② SPA + API

```text
CloudFront
    │
 ┌──┴──┐
 │     │
 ▼     ▼
S3    ALB
      │
      ▼
     API
```

例：
- `/` → S3
- `/api/*` → ALB

ReactやVueなどのSPAでよく採用される構成。

#### ③ SSR + 静的コンテンツ

```text
CloudFront
    │
 ┌──┴──┐
 │     │
 ▼     ▼
ALB    S3
```

例：
- `/` → ALB
- `/api/*` → ALB
- `/cdn/*` → S3

SSRアプリケーション（Next.jsなど）では画面表示をサーバーで生成するため、Web画面はALBへ転送する。一方で、動画や画像などの静的コンテンツのみS3へ振り分ける構成もよく採用される。

### 静的ファイルをキャッシュする方法

#### 方法① ALBをOriginのままキャッシュする

Behaviorを追加するだけでキャッシュ可能。

例：
- `/_next/static/*`
- `/logo.png`
- `/favicon.ico`

```text
Browser
    ↓
CloudFront（キャッシュ）
    ↓
ALB
```

構成を変えずに導入できるため、(IMO) 小〜中規模のシステムではよく採用される。

#### 方法② 静的ファイルをS3へ移す

```text
CloudFront
    │
 ┌──┴────────┐
 │           │
 ▼           ▼
ALB         S3
 │
 ▼
Next.js
```

例：
- `/` → ALB
- `/api/*` → ALB
- `/_next/static/*` → S3
- `/images/*` → S3

Next.jsでは`/_next/static`のファイル名にハッシュが付くため、長期間キャッシュしやすい。(IMO) 大規模サービスではこちらの構成が多い。

### ポイント
- (IMO) CloudFrontはシステムの入口として利用されることが多い。
- URLパス（Behavior）によってOriginを振り分ける。
- APIは通常キャッシュしない。
- 静的コンテンツは積極的にキャッシュする。
- OriginがALBでもCloudFrontでキャッシュできる。
- OACを利用するとS3をCloudFront経由でのみ公開できる。
