---
title: "環境変数の渡し方 - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > 環境変数の渡し方


## 基本方針
* (参考) https://qiita.com/KEINOS/items/518610bc2fdf5999acf2
* イメージに大事な値（機密情報）を書き込まないようにし、コンテナを起動する際に環境変数として値を渡すようにするのが基本(IMO)。

## Dockerfileで指定する方法

### 1. ENVで直接記載する
* (?) イメージに埋め込まれるため、機密情報を書くのは避けたほうがよい。この値は `-e` や `--env` で上書きできる。
    ```dockerfile
    ENV HOGE='FUGA' PIYO='PIYO PIYO'
    ENV hoo bar
    ```
* `ENV my_path=$PATH` のように書いても、ホスト側の環境変数は読み込まれず、イメージ内OSの環境変数が読み込まれるだけである点に注意。Dockerfileはホスト側の環境変数を直接呼び出せない。

### 2. --build-argで渡す
* (?) こちらもイメージに埋め込まれるため機密情報には向かない。Gitでタグ付けしたバージョン情報を渡したい場合などに使う。
    ```
    hoge='fuga'
    docker build --tag my_alpine --build-arg my_app_version=$hoge .
    ```

### 3. 外部シェルファイルをCOPYして呼び出す
* ENTRYPOINTやCMDで外部シェルファイルを呼び出し、その中で環境変数をexportする方法。
* (?) この方法はシェルスクリプト自体がイメージに書き込まれるため、大事な内容は書かないよう注意。docker-composeのようにビルド時点で外部ファイルから直接Dockerfileへ値を渡す方法はない。

## docker runで指定する方法

### 4. --env（-e）で起動時の引数として渡す
* ビルド時ではなく起動時に設定されるため、イメージには書き込まれないはず(IMO)。
    ```
    export MY_SECRET='MySecretIsHimitsu'
    docker run --rm \
      --env HOGE="fuga" \
      --env foo="bar" \
      --env MY_SECRET \
      alpine \
      env
    ```

### 5. --env-fileで渡す
    ```
    docker run --rm \
      --env-file ./my_env_file.txt \
      alpine \
      env
    ```
* (IME) `--env-file` で渡した値をシェル上で確認しようとすると、ホスト側の環境変数によって意図せず上書きされることがある。
    * 下記は `--env-file` で渡した値がそのまま出力される。
        ```
        docker run --env-file ./.env alpine echo $DOCKER_PREFIX
        ```
    * 一方で下記は出力されない（ホスト側の環境変数で上書きされてしまうようだ）。
        ```
        docker run --env-file ./.env alpine env
        ```
    * 下記のようにコンテナ内で評価させると、期待通り出力される。
        ```
        docker run --env-file ./.env alpine sh -c 'echo "$MEMO_DB_USER"'
        ```
    * `echo $VAR` の形はホスト側のシェルが変数展開してしまうため、コンテナに渡した環境変数を確認したい場合は `sh -c 'echo "$VAR"'` のようにコンテナ内で評価させる必要がある。

## docker composeで指定する方法
* compose file上での `environment` / `env_file` キーの詳細な構文は[Docker Compose](./docker_compose.md)を参照。
* `docker compose run` の引数として `-e` で渡すこともできる。docker-compose.yml側で `environment` に宣言しておく必要はない。
    ```
    docker compose run \
      --rm \
      -e HOGE=FUGA \
      -e foo=bar \
      サービス名 \
      env
    ```
* docker-compose.ymlの `environment` にキーだけ設定しておくと、ホスト側のシェルで定義されている環境変数がそのまま渡される。
    ```yaml
    services:
      my_app:
        image: alpine:latest
        environment:
          HOGE: FUGA
          foo: bar
          MY_SECRET:
    ```
    ```
    export MY_SECRET='MySecretIsHimitsu'
    docker compose run --rm my_app env
    ```
    * (?) `environment` に値も入れると、そちらが優先されるようだ。

## 環境変数の優先順位
* (参考) https://docs.docker.com/compose/environment-variables/#the-env-file
1. Compose file
2. Shell environment variables（docker composeを実行している環境のシェルで定義されている環境変数）
3. Environment file
4. Dockerfile
5. 未定義

`docker compose run` 実行時に `-e XXXX=YYYY` で指定した場合は、シェルの環境変数が上書きされるため、上記の2番と同様の扱いになると考えられる(IMO)。
* (?) docker composeのインストール方法によって、上記2番の優先順位の扱いが変わる場合があるという情報もある。
    * (参考) https://blog.brains-tech.co.jp/entry/2017/12/01/170600
