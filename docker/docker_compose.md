---
title: "Docker Compose - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > Docker Compose


## 概要
* 現在はdocker composeで従来のdocker-composeと同じコマンドが使えるため、docker-composeを別途インストールする必要はなくなっている(?)。
* 複数のコンテナから成るサービスを従来より簡単に管理できるようにしたオーケストレーションツール。類似のツールにVagrantがある。
* 従来は `docker run` コマンドに多くのオプションを付け、起動順序を意識したシェルスクリプトを書く必要があった（終了・再起動・削除も同様）。docker composeでは、これらをdocker-compose.ymlで宣言的に管理できる。

## 基本コマンド

### build
* YAMLに `build:` があれば、そのイメージをまとめてビルドする。
* `--force-rm`: 常に中間コンテナを削除する。`--rm` はビルド失敗時には削除しないが、こちらはビルド失敗時も含め常に削除する。基本的に付けておいたほうがよい(IMO)。
* `--no-cache`: 構築時にイメージのキャッシュを使わない。たまに使ったほうがよい場面がある。
* `--pull`: 常に新しいバージョンのイメージ取得を試みる。
* `--build-arg key=val`: サービスに対してビルド時の変数（args）を設定する。Dockerfileに渡すことができる。
    * 例: `docker compose build --build-arg username="my-user" --build-arg password="my-pass"`
    * これはdocker-compose.ymlの `args:` で指定するのとほぼ等価(IMO)。
        ```dockerfile
        FROM nginx:1.13
        RUN apt-get -y update && apt-get install -y \
            apache2-utils && \
            rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
        ARG username
        ARG password
        RUN htpasswd -bc /etc/nginx/.htpasswd $username $password
        ```

### images
* YAMLに `image:` があれば、そのイメージをまとめてプルする。

### up
* デフォルトは「アタッチド」モードで、全コンテナのログが画面上に表示される。「デタッチド」モード（`-d`）では、Composeはコンテナを実行すると終了するが、コンテナはバックグラウンドで動き続ける。
* docker-compose.ymlで定義したサービスの開始・再起動を行う。
    * `docker compose up -d redmine` のようにサービス名を個別指定することも可能。依存関係があるサービスも合わせて起動する（この例ではredmineとmysqlが依存関係にある場合、両方起動する）。
* イメージが作成されていなければ、イメージを作成してからコンテナを作成・起動する。
* コンテナが既に存在する場合は、イメージ・コンテナの再作成を行わず、（停止中の）コンテナを起動するだけになる。
    * (IME) `.env` を変更した場合など、明示的に `stop` しなくても `up -d` だけで反映されることがある。docker-compose.ymlの実行時に評価される項目については、ある程度追随してくれるようだ(?)。
* オプション
    * `-d`: デタッチド・モード。バックグラウンドでコンテナを実行する。`--abort-on-container-exit` と同時に使えない。
    * `--force-recreate`: 設定やイメージに変更がなくても、コンテナを再作成する。`--no-recreate` と同時に使えない。
    * `--no-recreate`: コンテナが既に存在していれば再作成しない。`--force-recreate` と同時に使えない。
    * `--no-build`: イメージが見つからなくても構築しない。
    * `--build`: コンテナ開始前にイメージを構築する。
    * `--abort-on-container-exit`: コンテナが1つでも停止したら全てのコンテナを停止する。`-d` と同時に使えない。
    * `-t, --timeout TIMEOUT`: アタッチしている、あるいは既に実行中のコンテナを停止する際のタイムアウト秒数を指定する（デフォルト: 10）。
    * `--remove-orphans`: Composeファイルで定義されていないサービス用のコンテナを削除する。

### ps
* 一覧を表示する（`docker ps` と異なりコンテナIDは表示されない）。

### restart
* 起動中のコンテナに対して `docker compose up -d` した場合、ソースコードに変更があれば取り込んで再起動する。ただし変更がない場合は「up to date」と表示されるだけで何も起こらない。
* `restart` の場合は変更の有無にかかわらず必ず再起動する。

### exec サービス（コンテナ名） /bin/bash
* 「サービス（コンテナ名）」はdocker-compose.ymlで指定している名前。
* ttyのないデーモンプロセス（cronなど）から実行する場合は、ttyがないというエラーになるため `-T` を付ける必要がある。
    * (参考) CDKからの実行もtty出力ではないため、そのままだとエラーになる。 https://github.com/docker/compose/issues/7306
* 例: `docker compose exec コンテナID /bin/bash` で直接コンテナに入ることができる。

### logs
* 関係するコンテナすべての出力を表示する。
* `docker compose logs サービス名` で指定のサービスのみ出力できる。

### stop
* 関係するコンテナをまとめて終了する。

### rm
* 関係するコンテナをまとめて削除する。

### create
* コンテナを作成する。

### start
* `start [サービス...]`
* 既存のコンテナをサービスとして起動する。オプションはない。使用頻度は低い(IMO)。

### run
* (?) `docker compose build` してから `docker compose run サービス名` とすると、意図した挙動にならないことがある。詳細未検証。
    * (参考) https://docs.docker.jp/v17.06/compose/reference/run.html

## docker-compose.ymlファイル
* (参考) https://docs.docker.jp/compose/compose-file.html
* (参考) https://futureys.tokyo/lets-understand-contents-of-docker-compose-yml/

### プロジェクト名とコンテナ名・volume名

#### プロジェクト名の振る舞い
* `COMPOSE_PROJECT_NAME` で指定できる。指定しない場合は、デフォルトでディレクトリ名がプロジェクト名として使われる。
* ディレクトリが変わると別のプロジェクトとして振る舞ってしまう。コンテナ名が同じでもプロジェクトが異なる場合、
    * `docker compose ps` には出てこなくなる。
    * `docker compose up` では新しいコンテナを作成しようとする。
* 既に作成済みのコンテナが属するプロジェクト名を後から変更する方法は見当たらなかった(?)。別プロジェクト名として立ち上げ直すのは、`COMPOSE_PROJECT_NAME` を変更するのとほぼ同じ挙動になる。
    ```
    docker compose -p プロジェクト名 up -d
    ```
* (参考) https://shinkufencer.hateblo.jp/entry/2022/03/19/000000

#### コンテナのName
* コンテナの「Name」は `(プロジェクト名)-(サービス名)-連番` という命名方式になる。プロジェクト名はデフォルトでフォルダ名が使われる。
* (参考) https://shinkufencer.hateblo.jp/entry/2022/03/19/000000
* (参考) https://qiita.com/okashoi/items/1ddf3724ad5c166e417b
* フォルダ名が同じ別プロジェクトでdocker composeを実行すると警告が出る。`docker compose up -d --remove-orphans` を実行すると、既に存在する `docker_(サービス名)_連番` のコンテナが全て削除されるので注意。
    ```
    WARNING: Found orphan containers (docker_nginx_1, docker_mysql_1, docker_echo_server_1, docker_php_fpm_1, docker_redis_1) for this project.
    ```

#### container_name（Compose File v2からの機能）
* `container_name` でコンテナ名を指定できる。
    ```
    container_name: ${DOCKER_PREFIX}_api
    ```

#### プロジェクト名とコンテナ名・volume名の名前空間
* コンテナ名は、プロジェクトが違っていても名前空間が分離されない(IMO 挙動としてはやや扱いにくい)。そのため、別プロジェクトで同名のコンテナを立ち上げるとエラーになる。
* volume名はプロジェクトに依存しないようだ(?)。`external: true` を付けると、同名の既存volumeが存在する場合はそれを使うようになる。

### image
* タグやイメージIDの一部を指定する。ローカル・リモートどちらでもよい。
* ローカルに存在しなければ、Composeがイメージを取得（pull）する。
* イメージの検索: https://hub.docker.com/search?q=&type=image
* `image` と `build` を同時に宣言している場合（Compose File v2からの機能）は、ビルドが優先的に実行され、`image` はその成果物のタグ付けに使われる（`image` によるpullは発生しない）。結果としてイメージ名は `image` で指定したものになる。
    * デフォルトだとイメージ名が長くなりがちなので、この使い方は便利(IMO)。タグの付与も `image` から可能。
    * (参考) https://amaya382.hatenablog.jp/entry/2017/04/03/034002

### ports
* 公開用のポート。ホスト側・コンテナ側の両方を `ホスト側:コンテナ側` の書式で指定できるほか、コンテナ側のポートのみも指定できる。
* `ホスト側:コンテナ側` の書式でポートを割り当てる際、コンテナのポートが60以下だとエラーになる場合がある。YAMLが `xx:yy` 形式の指定を60進数の数値とみなしてしまうため(?)。そのため、ポートの割り当ては常に文字列として指定することが推奨される。

### expose
* ホストマシン上で公開するポートを指定せず、コンテナが公開するポート番号のみを指定する。リンクされたサービス間でのみアクセス可能になる。内部で使うポートのみ指定できる。
    ```yaml
    expose:
      - "3000"
      - "8000"
    ```

### 実行シェル上の環境変数の参照
* `.env` ファイルに環境変数を入れておくと、docker-compose.ymlはデフォルトでそれを読み込む。docker-compose.ymlファイル内でその環境変数を利用できる。
    ```yaml
    image: envtest-node:${IMAGE_VERSION:-invalid}
    ```
    このように参照でき、`IMAGE_VERSION` が存在しない場合は `invalid` になる。

### env_file
* コンテナ内で使う環境変数をファイルから設定できる。設定したファイルの内容が、コンテナ内で環境変数として展開される（コンテナ内で `env` コマンドを実行すると確認できる）。
* `environment` で個別に上書きできる。
* 相対パス・絶対パスどちらでも指定できる。
    ```yaml
    env_file:
      - .env
      - ./common.env
      - ./apps/web.env
      - /opt/secrets.env
    ```
    `docker run --env-file=FILE ...` に近い動作(IMO)。
* 再評価のタイミング
    * `restart` では再評価されない。
    * `up -d` では再評価される。
    * (IMO) アプリケーション側で `env_file` 経由でコンテナに読み込んだ環境変数を参照するのではなく、ファイルを直接参照する設計にしておくと、dockerの再起動なしで環境変数の変更を反映しやすくなる場合がある（その場合、アプリ側での再読み込み処理は別途必要）。

### build

#### args
* (参考) https://blog.cloud-acct.com/posts/u-env-docker-compose/
* Dockerイメージをビルドする際に引数を渡す。つまり、docker-compose.ymlからDockerfileへ環境変数を渡すことができる。
    ```yaml
    services:
      api:
        build:
          context: ./api
          args:
            # キー: 値
            WORKDIR: $WORKDIR
            # この書き方でもよい
            - WORKDIR=$WORKDIR
            - buildno=1
            - user=someuser
    ```
    Dockerfile側
    ```dockerfile
    ARG WORKDIR
    ENV HOME=/${WORKDIR}
    ```

#### target
* Dockerfileで指定している `AS 〜` を参照する。
* (参考) https://qiita.com/dfurusaka/items/24263b8194cdf0af2298

### environment
* 環境変数を追加する。配列もしくは辞書形式（dictionary）で指定できる。
* キーだけの環境変数は、Composeの実行時にマシン上で指定するものであり、シークレット（API鍵などの秘密情報）やホスト固有の値を指定するのに便利。
* Dockerfile・`docker run`・composeそれぞれでの環境変数の渡し方の比較や優先順位については[環境変数の渡し方](./docker_env.md)を参照。

### context
* `docker build` コマンドを実行したときのカレントなワーキングディレクトリを、ビルドコンテキスト（build context）と呼ぶ。
* 範囲は `.` のように広く取らず、適切な範囲に設定しておいたほうがよい。
    * (IME) 各コンテナでvolumeをマウントしていると、運用開始後にbuildした際、読み込みでpermissionエラーになることがある。そのため、最初の時点でcontextの必要範囲を最低限に絞っておいたほうがよい。

### volume

#### PostgreSQLの例
```yaml
volumes:
  - db-volume:/var/lib/postgresql/data
  - ./initdb/db:/docker-entrypoint-initdb.d # 初期化が不要なら入れなくてよい

volumes:
  db-volume:
    name: ${DOCKER_PREFIX}_volume
```

### composeファイルの上書き
* (参考) https://qiita.com/hoto17296/items/a8a85d5244f46c119278
    ```
    COMPOSE_FILE=docker-compose.yml:docker-compose.override.yml
    ```
* こうしておくと、overrideファイルの内容で上書きされる。

### anchor（`&`）で設定を再利用する
* `&アンカー名` で定義した設定を `*アンカー名` で再利用できる。
* (参考) https://techracho.bpsinc.jp/hachi8833/2020_02_07/87447
    ```yaml
    logging: &log
      # ...
    logging: *log
    ```

### logging
* コンテナのPID=1の標準出力（エラー出力も含む）は `docker container logs コンテナ名` 等で確認できるが、これはDockerの標準機能で `/var/lib/docker/containers/` にコンテナ内のログが格納される仕組みによるものである。そのため、Linuxのlogrotate等を別途使用する必要はない。
* サイズを指定することもできる。
    * (参考) https://qiita.com/katsuNakajima/items/1828f570a88ab3e5a0e6
    ```yaml
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    ```
* AWSのCloudWatchへ格納することも可能。
    * (参考) https://blog.uni-3.app/2017/11/24/docker-log-to-aws-cloudwatch
    ```yaml
    logging:
      driver: "awslogs"
      options:
        awslogs-region: "ap-northeast-1"
        awslogs-group: "docker" # group name
        tag: '{{ with split .ImageName ":" }}{{join . "_"}}{{end}}-{{.ID}}' # stream nameの名前 "image名_tag-ID"
        awslogs-create-group: "true" # groupがなければ作成される
    ```

### profile
* profileが付いているコンテナは、`docker compose up` の対象から除外できる。
* (参考) https://gotohayato.com/content/505/

### command
* 複数行で書く場合は `command: bash -c "〜 && 〜"` のようにする。bashを使わないと `&&` などが使えないため。
