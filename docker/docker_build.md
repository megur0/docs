---
title: "Dockerfile・ビルド - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > Dockerfile・ビルド


## Dockerfile
* `FROM`: ベースとなるイメージ
* `MAINTAINER`: 作成者の情報。(?) 現在は非推奨で、代わりに `LABEL` の使用が推奨されている。
* `RUN`: コマンドの実行（イメージ作成時に実行される）
* `ADD`: ファイル・ディレクトリの追加
* `CMD`: コンテナの実行コマンド（1つのみ有効）
* `ENTRYPOINT`: コンテナの実行コマンド
    * `docker run` やdocker-composeの `command` で上書きできるのは `CMD` のみで、`ENTRYPOINT` は上書きできない点が異なる。
    * (参考) https://penpen-dev.com/blog/docker-cmd-entrypoin/
* `WORKDIR`: 作業ディレクトリの指定（存在しない場合は作成される）
* `COPY <コピー元> <コピー先>`
    * コピー先はWORKDIRからの相対位置。
    * `<コピー元>` で指定したファイル・ディレクトリを、イメージのファイルシステム上の `<コピー先>` に追加する。
    * `COPY hom* /mydir/` のように、「hom」で始まるファイルすべてを追加することもできる。
    * 注意: `COPY ./app ./` のように書くと、`./app` の中身が `./` へ展開される。`app` ディレクトリごとコピーしたい場合は `COPY ./app ./app` とする。
* `ENV`: 環境変数の指定
* `ARG`: Dockerfile内で使用できる変数。`ENV` と異なり、Dockerfile内でのみ使用できる。
    * `ADD`, `COPY`, `WORKDIR`, `RUN`, `USER`, `ENV` などで使える。
    * 例
        ```dockerfile
        FROM busybox
        ARG hoge  # ARG hoge="world" のようにデフォルト値も設定できる
        RUN echo $hoge
        ```
        ```
        docker build ./ --build-arg hoge=hello
        ```
    * デフォルト値は変数使用時に設定することもできる。
        ```dockerfile
        FROM busybox
        ARG user
        USER ${user:-some_user}
        ```
    * (参考) https://qiita.com/chimame/items/e959843e86419f51e45a
* `EXPOSE`: ポートのエクスポート
* `VOLUME`: ボリュームのマウント
* `USER`: 実行ユーザーの指定

## EXPOSEとdocker runの --expose, --publish-all（-P）, --publish（-p）
* (参考) https://qiita.com/kubocchi/items/dee7498ec2dabacc503f
* `EXPOSE`
    * ポートを公開するわけではなく、ドキュメンテーションとしての役割を持つ。
        * コンテナ内部で利用されているポートのうち、ホストマシンにバインドする意図のあるポートを `EXPOSE` に記述しておくことで、どのポートをpublishすればよいかが分かりやすくなる。
    * `--publish-all`（`-P`）を指定すると、EXPOSEしておいたポートをすべてホストマシンへ公開できる。
* `--expose`
    * 指定すると、Dockerfileで指定したEXPOSEに追加される。
* `--publish`（`-p`）
    * `-p 8080:80` のように指定すると、コンテナの80番ポートをホスト側の8080としてpublishできる。
    * Dockerfile側でEXPOSEされている必要はなく、`-p` オプションだけで済ませることもできる。

## ビルド
* (参考) https://knowledge.sakura.ad.jp/15253/
* 中間コンテナ
    * Dockerfileに書かれたコマンドは、毎回中間的なDockerコンテナとして起動し、各コマンドを実行して各段階でDockerイメージを作成する、という処理を繰り返す。この各段階のDockerイメージはレイヤーと呼ばれる。
    * そのため、一つ前のコマンドの実行状態がそのまま引き継がれるわけではない。例えば以下のような記述があった場合、`test.txt` は「/tmp」配下ではなく「/」配下に作成される。
        ```dockerfile
        RUN cd /tmp           # (1)
        RUN touch test.txt    # (2)
        ```
        (1)の処理は中間的なDockerコンテナ上で `cd /tmp` が実行されるが、(2)の処理は新しく起動したコンテナで実行されるため、結局「/」配下で実行される。
    * Dockerfileの各コマンドごとにレイヤーが作成されるため、その分Dockerイメージのサイズが大きくなる。なるべく1コマンドにまとめて実行することが推奨される。また、作成できるレイヤーには上限（128レイヤー）があるため、その意味でもまとめて実行したほうがよい。
        * 例えばyumインストールも、個別のyumコマンドで実行するのではなく、一度のyumコマンドで複数インストールするとよい。ただし、キャッシュとの兼ね合いもあるため、どこまでまとめて実行するかは検討が必要。
* キャッシュ
    * dockerビルド時、Dockerfileで既に実行済みのコマンドはキャッシュが使用される。そのコマンド以降は全て再実行されるため、よく変更するコマンドは後の方に書いたほうがよい。

## .dockerignore
* ビルド時に無視するファイルを指定する。何も指定しないと、ビルド時にコンテキスト内のファイルをすべて読みに行くため、ファイルが大量にある場合はビルドに時間がかかったり、permission errorになったりすることがある。

## COPYを使って他のイメージのモジュールを利用する
* composerのインストールを直接RUNで書くと、ハッシュ値（composerのインストールごとに変わる）の指定などが必要になり煩雑になる。
    * (参考) https://qiita.com/yatsbashy/items/02bbbebbfe7e5a5976bc
    ```dockerfile
    COPY --from=composer /usr/bin/composer /usr/bin/composer
    ```
* 同様にnodeも利用できる。
    ```dockerfile
    COPY --from=node /usr/local/bin/node /usr/local/bin/node
    COPY --from=node /usr/local/lib/node_modules/ /usr/local/lib/node_modules
    ```
