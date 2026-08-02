---
title: "Organizations・IAM Identity Center - AWS"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(AWS)](./README.md) > Organizations・IAM Identity Center


## この分野が分かりづらい理由
AWS Organizations、AWSアカウント(管理アカウント/メンバーアカウント)、IAM Identity Centerのユーザー・Permission Set・アカウントへの割り当て、といった要素が複数の階層にまたがっており、それぞれの役割が初見では掴みにくい(IME)。まず全体の関係図を押さえてから各要素を見ていく。

## 全体像

```
AWS Organizations
│
├─ 管理アカウント (Management Account)
│   … Organizationsを有効化した最初のAWSアカウント。組織全体の管理拠点。
│   │
│   └─ IAM Identity Center を有効化する場所
│        ├─ Users            … 実在する人物のログインID
│        ├─ Permission Sets  … 「何ができるか」を定義した権限テンプレート(IAMポリシーの集合)
│        └─ Account assignments
│             (どのUserを、どのAWSアカウントに、どのPermission Setで割り当てるか)
│
└─ メンバーアカウント (Member Account) ×N
     … Organizations配下に作成する個別のAWSアカウント。
       実際のEC2・RDS・S3などのリソースはこちらに構築する。
     │
     └─ Permission Setが割り当てられると、このアカウント内に
        自動でIAMロールが生成される(例: AWSReservedSSO_AdministratorAccess_xxxxxxxx)
        → 自分でIAMロールを作る必要はない
```

さらに、Identity Centerの「割り当て」は次の3つの掛け合わせだと考えると理解しやすい。

```
Identity Center User (例: taro.yamada)
        ×
Permission Set (例: AdministratorAccess)
        ×
対象アカウント (例: メンバーアカウント 111122223333)
        ↓
対象アカウント内に自動生成されるIAMロール
        ↓
aws sso login で取得する一時的なSTS認証情報
```

## なぜこの仕組みになっているのか(Why)
「AWSアカウント」「ルートユーザー/IAMユーザー」「Identity CenterのUser」と、ユーザーらしき概念が複数出てくるため混乱しやすい。それぞれが表しているものを整理する。

### ルートユーザー・IAMユーザーの位置づけ
* ルートユーザー、IAMユーザーは「AWSアカウント」という概念そのものではなく、AWSアカウントという**箱に従属する、その箱の中だけで通用する認証の主体(プリンシパル)**。
    * ルートユーザー: アカウント作成時に自動生成される、そのアカウントの持ち主を表す唯一無二の認証情報(メール+パスワード)。アカウントと一心同体で、他のアカウントには一切通用しない。
    * IAMユーザー: そのアカウント内で追加作成できる認証の主体。これも他のアカウントには通用しない。
* 「ユーザー」という名前ではあるが、正確には**リソースを操作するための「行為主体」**であり、「リソースを管理する単位(箱)」自体はAWSアカウント側が担う。
* IAMユーザーは必ずしも人間を表すとは限らない。CI/CDパイプラインやアプリケーションが使う機械的なアクセスキーの持ち主として作成されることもある(近年は非推奨の流れ)。

### AWSアカウントは「人」の単位ではない
* AWSアカウントはリソースと請求をまとめる箱。ログインはできるが、それは「箱に対応するID(ルート/IAMユーザー)」のログインであり、組織横断で人を表す概念ではない。
* AWSアカウント単体にはSSO機能が存在しない。「AWSアカウントに直接SSOログインする」という発想自体、実は成立しない。SSOという機能を提供しているのはIdentity Center側。

### Identity Centerは「人」を表す唯一の概念
* Identity CenterのUserは、組織全体で「人」を一意に表す唯一の概念。SSO機能もこちらが提供する。
* Userとアカウントを分離しているからこそ、1人が複数アカウントを扱う構成や、アカウント側を一切触らずに人のアクセスを一括制御する運用が可能になる。

### 人が増えた時・減った時の非対称性
* **人が増えた時**: Userの作成 + **対象アカウントへの割り当て(Assignment)**の2ステップが必要。作成しただけではどのアカウントにもアクセスできない。
* **人が減った時**: そのUserを無効化/削除する**だけ**で、割り当てていた全アカウントへのアクセスが一括で失われる。各アカウントを回って個別に消す必要がない。
* この「追加は明示的な割り当てが要るが、削除は1箇所で済む」という非対称性が、Identity Centerを使う一番のメリット。
* 対してIAMユーザー方式(アカウントごとに個別管理)だと、人が増えるたびにアクセスすべきアカウントの数だけIAMユーザーを作る必要があり、退職・異動時は全アカウントを回ってIAMユーザーを無効化しないと権限が残り続けるリスクがある。

### なぜAWSアカウント(メンバーアカウント)を分けるのか
* リソースを1つにまとめたいなら、メンバーアカウントは1つのままでよい。
* 分離したい理由があるなら、その単位でメンバーアカウントを増やす。実務でよくある分割基準は以下の通り。

|分割基準|例|
|-|-|
|環境ごと|dev / staging / prod|
|システム・プロダクトごと|自社システムA / 自社システムB|
|顧客・案件ごと|受託システムC(顧客ごとに分けることも)|
|役割ごと|ログ集約専用アカウント、セキュリティ監査専用アカウントなど|

分ける実利は以下の通り。

* **請求の分離**: システムごと・顧客ごとの利用料金を明確に分けられる
* **被害範囲の分離**: あるシステムの設定ミスや侵害が、他のシステムに波及しない
* **サービスクォータの分離**: AWSの各種上限(EC2起動数など)はアカウント単位なので、あるシステムが上限に張り付いても他システムに影響しない
* **契約・コンプライアンス上の要件**: 受託案件で「専用インフラであること」が契約条件になっている場合など

Identity Center側は、これらのアカウントが何個に増えても、Userと割り当てを追加するだけで対応できる。

## AWS Organizations

### 概要
* 複数のAWSアカウントを一つの組織としてまとめて管理する仕組み。統合請求(Consolidated Billing)、SCP(Service Control Policies)によるアカウント横断の権限制御などができる。
* 有効化(＝組織の作成)、メンバーアカウントの追加、統合請求、SCPの利用に追加料金は発生しない。無料。

### 管理アカウントとメンバーアカウント
* Organizationsを有効化したアカウントは「管理アカウント(Management Account)」になる。
* 管理アカウントには以下の性質があるため、**実運用リソースは置かず、組織運営専用にする**のがベストプラクティス。
    * 組織全体のSCPを管理でき、他の全メンバーアカウントに対して強い統制権を持つ
    * 統合請求の起点になる
    * 侵害された場合の被害範囲が組織全体に及ぶ
    * 一度管理アカウントになると、後から別アカウントに管理アカウントの立場を移す作業は通常のアカウント運用より手間がかかる(?)
* 実際にAWSコンソールで「AWS Organizations を有効にする」際、「このアカウントにはリソースを保存しないことを推奨する」という趣旨の警告が出るのはこのため。

### メンバーアカウントの作成
```
AWS Organizations → AWS accounts → Add an AWS account → Create an AWS account
```
* **AWS account name**: 用途が分かる名前(例: `terraform-workload`)
* **Email address**: そのAWSアカウント専用のルートメールアドレス。**AWS全体で他のどのアカウントにも使われていない一意なアドレス**が必要。
    * Gmail等プラスアドレス(`your-address+terraform@gmail.com`)が使えるメールサービスなら、既存アドレスの流用で新規メールボックスを用意せずに済む。
* **IAM role name**: 管理アカウントからこのメンバーアカウントへスイッチロールする際に使うロール名。デフォルトの`OrganizationAccountAccessRole`のままで問題ない。
* 作成には数分かかる。`AWS accounts`一覧で`Active`ステータスになれば完了。作成後、一覧画面からアカウント名をクリックすると12桁のAccount IDを確認できる。

### 費用について
* メンバーアカウントの作成自体に料金はかからない。
* 課金されるのは各アカウント内で実際に作成したリソース(EC2、RDS、S3等)のみ。アカウントを分けても合計額は基本的に変わらない。
* 統合請求により、複数アカウントの利用量が合算され、S3のリクエスト数割引やSavings Plans/リザーブドインスタンスの適用範囲が組織全体で共有される場合がある(組織側の設定次第)。
* AWS Free Tier(無料利用枠)の適用条件は見直しが入っている可能性があるため、新規メンバーアカウントでの適用状況はAWSコンソール上の最新の案内で確認するのが確実(?)。

## IAM Identity Center

### 概要
* IAM Identity Centerは「実在する人物」を表すためのID管理サービスで、**1つのIDで複数のAWSアカウントを横断して**アクセスできる。人間のログインにはこちらを使い、長期間有効なアクセスキーを持たずに済む点が主な利点。
* IAMユーザーとの違い・なぜIdentity CenterにUsersという概念が必要なのかは[なぜこの仕組みになっているのか(Why)](#なぜこの仕組みになっているのかwhy)を参照。
* IAM Identity Centerを有効化するにはAWS Organizationsが前提となる(単体アカウントの場合は有効化時にOrganizationsが自動作成される)。

### 用語整理
|用語|役割|
|-|-|
|User|Identity Center上のログインID。実在する人物に対応する|
|Permission Set|「何ができるか」を定義した権限テンプレート(IAMポリシーの集合に相当)|
|Assignment(割り当て)|User × Permission Set × AWSアカウントの組み合わせを紐付ける操作|

### ユーザーの作成
```
IAM Identity Center → Users → Add user
```
* Username、Email address、First name/Last nameなどを入力。
* **Usernameは作成後に変更できない**。役割名(`admin`等)ではなく、個人名ベース(例: `taro.yamada`)にしておくのが無難(IMO)。
    * 理由: CloudTrailの監査ログにこのユーザー名がそのまま記録されるため、後から人を追加した際に「誰が何をしたか」が名前で追いやすい。
* Emailは後から変更できる可能性が高い(?)。実際に受信できるメールアドレス(初回パスワード設定・パスワードリセット等が届く)を指定する。このメールアドレスはAWSアカウント作成時のルートメールアドレスとは別物で、AWS全体で一意である必要はなく、Identity Centerインスタンス内で一意であればよい。
* 作成後、対象ユーザーのメールアドレスに招待が届き、パスワード設定を行う。

### Permission Setの作成
```
IAM Identity Center → Permission sets → Create permission set
```
* Permission set type:
    * `Predefined permission set`(AWS管理ポリシーから選択。例: `AdministratorAccess`)
    * `Custom permission set`(独自ポリシーを組み合わせ)
* 個人開発・Terraform用途であれば、まず`AdministratorAccess`で作成しておくのが簡単(IMO)。
* Session duration(セッション有効時間)もここで設定できる(デフォルト1時間、最大12時間まで延長可能)。

### アカウントへの割り当て
```
IAM Identity Center → AWS accounts → 対象アカウントを選択 → Assign users or groups
```
* 割り当てるユーザーとPermission setを選択する。
* **管理アカウントには割り当てず、メンバーアカウントに対して割り当てる**(管理アカウントに強い権限を持たせないため)。
* 割り当てを行うと、対象アカウント内に自動でIAMロールがプロビジョニングされる。自分でIAMロールを作成する必要はない。

## メンバーアカウントへのアクセス方法
Organizationsで作成したメンバーアカウントは、IAMユーザーではなく**独立したAWSアカウント**([なぜAWSアカウント(メンバーアカウント)を分けるのか](#なぜawsアカウントメンバーアカウントを分けるのか)参照)。アクセス方法は主に3つある。

### 1. OrganizationAccountAccessRoleでロール切り替え(AssumeRole)
メンバーアカウント作成時、そのアカウントには通常以下のIAMロールが自動作成される([メンバーアカウントの作成](#メンバーアカウントの作成)の`IAM role name`参照)。

```
OrganizationAccountAccessRole
```

管理アカウント側で十分な権限を持つ主体(IAMユーザーやロール)から、このロールへAssumeRole(コンソールでは「Switch Role」)することでメンバーアカウントにアクセスできる。

```
Management Account
    ↓ AssumeRole
OrganizationAccountAccessRole
    ↓
Member Account
```

Terraform運用や、管理アカウント側からの日常的な管理作業では、この方法が最も一般的([Terraformでのマルチアカウント運用](#terraformでのマルチアカウント運用)を参照)。

### 2. IAM Identity Center(SSO)経由でログイン
人間がブラウザでログインする場合、AWSが推奨しているのはこちらの方法([AWSコンソール(ブラウザ)へのログイン](#awsコンソールブラウザへのログイン)を参照)。1つのUserで複数アカウントを横断でき、アカウントごとにIAMユーザーを作らずに済む。

### 3. ルートユーザーでログイン(緊急時のみ)
技術的には可能だが、Organizationsで作成したメンバーアカウントには**初期パスワードが設定されていない**。ログインするには以下の手順でパスワードを設定する必要がある。

1. メンバーアカウント作成時に指定したメールアドレスを確認する([メンバーアカウントの作成](#メンバーアカウントの作成)の`Email address`)
2. AWSサインイン画面で「パスワードをお忘れですか?」を実行
3. 届いたメールの案内に従ってルートパスワードを設定
4. ルートユーザーとしてログイン

ルートユーザーは全権限を持ちアカウント個別の管理になるため、緊急時以外は利用しないのがAWSのベストプラクティス([管理アカウントとメンバーアカウント](#管理アカウントとメンバーアカウント)も参照)。

### (非推奨) メンバーアカウントにIAMユーザーを作成してログイン
`https://<アカウントID>.signin.aws.amazon.com/console`から専用IAMユーザーでログインする方法もあるが、これは「アカウントごとに個別のIDを持つ」という、Identity Centerで避けたい運用に逆戻りしてしまう(IMO)。

## AWSコンソール(ブラウザ)へのログイン
[AWS access portal URL](#aws-configure-sso)(`https://xxxxx.awsapps.com/start`形式)にブラウザでアクセスし、Identity Centerのユーザーでログインすると、CLIで使っているのと同じ「アカウント×Permission Setの一覧」画面が表示される。各アカウント/ロールの行に2つの選択肢がある。

* **Management console**: クリックするとそのままブラウザで対象アカウントのAWSコンソールが開く(パスワード入力不要、フェデレーションされたセッションとしてログイン)
* **Command line or programmatic access**: CLI用の一時credentials表示(こちらが`aws sso login`で使っている経路)

つまり「CLIでの`aws sso login`」と「コンソールでの`Management console`クリック」は、同じ認証基盤の入り口違いなだけで、裏側の仕組みは同じ。ログインしているのはIdentity Centerのユーザーであり、対象アカウント内には自動生成されたIAMロール(`AWSReservedSSO_AdministratorAccess_xxxxxxxx`)にフェデレーションでスイッチしている形になる。

## Terraformでのマルチアカウント運用
環境ごとにメンバーアカウントを分ける構成([なぜAWSアカウント(メンバーアカウント)を分けるのか](#なぜawsアカウントメンバーアカウントを分けるのか)も参照)は、Terraform運用でもよくあるパターン。

```
Management Account
├─ dev
├─ stg
└─ prod
```

管理アカウントから、各環境アカウントの`OrganizationAccountAccessRole`(または環境ごとに専用に作成したロール)へAssumeRoleする形で運用する。

```
Management Account
    ↓ AssumeRole
dev

Management Account
    ↓ AssumeRole
stg

Management Account
    ↓ AssumeRole
prod
```

Terraform自体も、各アカウントのロールをAssumeRoleしてリソースを管理するのが一般的(例: `provider "aws" { assume_role { role_arn = "arn:aws:iam::<各アカウントID>:role/OrganizationAccountAccessRole" } }`のような設定)。

## CLIでの利用

### aws configure sso
```bash
aws configure sso
```
### 対話内容はCLIバージョンによって異なる
`sso-session`(推奨形式、トークン自動リフレッシュ対応)は AWS CLI **v2.9系以降**で使える機能。実際に検証した結果、以下の違いを確認した。

|バージョン|挙動|
|-|-|
|v2.7.30|`SSO session name`という質問自体が出ない。常にレガシー形式(単一`[profile]`ブロック)で書き込まれる|
|v2.9.6以降|`SSO session name (Recommended)`が質問される。名前を入力すると`sso-session`ブロックに分離された推奨形式になり、空欄で進めるとレガシー形式になる|

つまり「推奨形式にならない」場合、まず`aws --version`でバージョンを確認し、v2.9系より古ければアップデートする(`brew upgrade awscli`、または[インストール手順](./aws_cli_iac.md)のpkgインストーラーを再実行)のが対応策になる。実際にv2.7.30からv2.36.14へアップデートしたところ、`SSO session name`の質問が表示されるようになることを確認済み。

新しいバージョン(v2.9系以降)での対話例:

* `SSO session name`(推奨): 任意の名前。空欄のまま進めるとレガシー形式になる。
* `SSO start URL`: 以下のいずれかを入力する。どちらも`IAM Identity Center → Dashboard`の`Settings summary`(または`Settings → Identity source`)に表示されている。
    * **AWS access portal URL**(例: `https://xxxxx.awsapps.com/start`): ブラウザで直接開いてログインするための人間向けポータル。アカウント作成時の招待メールにも記載されている。
    * **Issuer URL**(例: `https://identitycenter.amazonaws.com/ssoins-xxxxxxxxxxxxxxx`): CLIのOIDC認証には問題なく使えるが、**ブラウザで直接開いてもログインポータルにはならない**。
* `SSO region`: Identity Centerを有効化したリージョン
* `SSO registration scopes`: デフォルト(`sso:account:access`)のままでよい

ブラウザが開いてログイン後、割り当てられたアカウント/ロールの一覧から選択し、region/output/プロファイル名を入力すると`~/.aws/config`に設定が書き込まれる。

**推奨形式(SSOトークンプロバイダー設定)**: セッション名を入力した場合(v2.9系以降のみ)。`sso-session`ブロックが分離され、複数プロファイルでのセッション共有やトークンの自動リフレッシュに対応する。
```ini
[profile myprofile]
sso_session = my-sso
sso_account_id = 111122223333
sso_role_name = AdministratorAccess
region = ap-northeast-1
output = json

[sso-session my-sso]
sso_region = ap-northeast-1
sso_start_url = https://xxxxx.awsapps.com/start
sso_registration_scopes = sso:account:access
```

**レガシー形式(非リフレッシュ設定)**: セッション名を空欄で進めた場合、またはv2.9系より古いCLIの場合。単一の`[profile]`ブロックに直接書き込まれ、トークンの自動リフレッシュには対応しない。今のところ機能が廃止されたわけではなく、単に古いバージョンや未入力時の挙動として現役でサポートされている。
```ini
[profile myprofile]
sso_start_url = https://identitycenter.amazonaws.com/ssoins-xxxxxxxxxxxxxxx
sso_region = ap-northeast-1
sso_account_id = 111122223333
sso_role_name = AdministratorAccess
region = ap-northeast-1
output = json
```

古いCLIのままレガシー形式を使い続けても、個人利用でプロファイルが1つだけなら実用上の支障はない(IMO)。推奨形式にしたい場合はCLIのアップデートが必要。

いずれの形式・いずれの`sso_start_url`の値でも、`aws sso login`/`aws sso logout`の挙動やCLIでの利用方法は変わらない。ただし**ブラウザでAWSコンソールにログインしたい場合は、`sso_start_url`の値ではなく、必ずAWS access portal URL(`*.awsapps.com/start`形式、招待メールにも記載)を使う**こと。`sso_start_url`にIssuer URLが設定されている場合、そのURLをブラウザで直接開いてもログイン画面は表示されない。

(出典: [Configuring IAM Identity Center authentication with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)、[Customizing the AWS access portal URL](https://docs.aws.amazon.com/singlesignon/latest/userguide/howtochangeURL.html))

### 設定(config)とセッション(トークン)の違い
|対象|保存場所|ログイン/ログアウトの影響|
|-|-|-|
|プロファイル設定(start URL、region、account ID、role名など)|`~/.aws/config`|影響なし。恒久的に残る|
|一時的な認証トークン(セッション)|`~/.aws/sso/cache/*.json`|ここだけがリセット/発行される|

`aws sso login`/`aws sso logout`は上記のうちセッションのトークンだけを操作し、`~/.aws/config`の内容は一切書き換えない。そのため、ログアウト→ログインを繰り返してもURLやリージョンを毎回入力し直す必要はなく、ブラウザ側でのログイン(場合によってはパスワード入力)のみで済む。

### ログイン・ログアウト
```bash
aws sso login --profile myprofile
aws sso logout --profile myprofile
```
* `login`はセッションが既に有効な場合は再認証をスキップする。
* `logout`はキャッシュされたトークンを破棄する。`--profile`を省略すると、ローカルにキャッシュされている全SSOセッションをまとめてログアウトする。

### セッション状態(ログイン状況)の確認
`aws configure list`はプロファイルの静的な設定内容を表示するだけで、SSOセッションが有効かどうかまでは分からない。セッション状態の確認には以下を使う。

```bash
aws sts get-caller-identity --profile myprofile
```
* セッションが有効ならAccount/Arnが返る。
* セッションが切れていると以下のようなエラーになり、これが実質的な「ログイン状態チェック」として機能する。
```
The SSO session associated with this profile has expired or is otherwise invalid. To refresh this SSO session run aws sso login with the corresponding profile.
```

有効期限を直接確認したい場合は、キャッシュされたトークンファイルを見る方法もある(やや裏技的)。
```bash
cat ~/.aws/sso/cache/*.json
```
`expiresAt`フィールドに有効期限(UTC)が入っている。

### `--profile`を省略する
対象アカウントが実質1つだけの個人利用であれば、該当プロファイルをデフォルトプロファイルにするのが簡単(IMO)。`~/.aws/config`の`[profile myprofile]`ヘッダーを`[default]`に書き換えるだけでよい(中身の各設定値はそのまま)。

```ini
[default]
sso_start_url = https://xxxxx.awsapps.com/start
sso_region = ap-northeast-1
sso_account_id = 111122223333
sso_role_name = AdministratorAccess
region = ap-northeast-1
output = json
```

(`[sso-session]`ブロックを使っている形式の場合は、そのブロックはそのまま残し、`[profile]`側のヘッダーだけを`[default]`に書き換えればよい。)

これで以下のように`--profile`無しで操作できる。
```bash
aws sso login
aws sts get-caller-identity
aws s3 ls
aws sso logout
```

将来、管理アカウント用プロファイルなど複数プロファイルを使い分ける予定があるなら、`[default]`にはせず`export AWS_PROFILE=myprofile`を`~/.zshrc`に書く方法のほうが、プロファイルの切り替えミスを防ぎやすい(IMO)。

## 参考: 今回行った作業の流れ

```
1. ルートアカウントでログイン中のAWSアカウントで Organizations を有効化
   → このアカウントが「管理アカウント」になる(リソースは置かない)

2. Organizations でメンバーアカウントを新規作成
   → Terraformで実際に使うリソースはこちらに構築する

3. IAM Identity Center を有効化し、User を作成(個人名ベースのUsername)

4. Permission Set(AdministratorAccess)を作成

5. メンバーアカウントに対して User × Permission Set を割り当て

6. ローカルで aws configure sso → aws sso login
   → メンバーアカウントに対する一時的な認証情報を取得
```
