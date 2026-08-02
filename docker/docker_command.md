---
title: "コマンド - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > コマンド


## docker -h

## build
* Dockerfileから、イメージを作成する。
    * `docker build [ -t ｛イメージ名｝ [ :｛タグ名｝ ] ] ｛Dockerfileのあるディレクトリ｝`
        * `-t` オプションは作成するDockerイメージのイメージ名およびタグ名を指定する。
    * 例: `docker build -t myname/nginx:1.0 .`

### マルチステージビルド
* 例えば最終的には実行環境だけのイメージが欲しい場合など、中間イメージにビルド処理を任せることでイメージを小さくできる。
* 最終的なイメージは最後の `FROM` 命令で指定したベースイメージをもとに生成される。
    * (参考) https://maku77.github.io/p/z3n4hye/

## タグ付けに関するメモ
* イメージを全て削除した後に `docker compose build` すると、下記のようなイメージが作成される。
    ```
    REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
    aws-deploy_nginx     latest    58616a6c32d1   8 minutes ago   151MB
    aws-deploy_php_fpm   latest    64ea1bcb4963   8 minutes ago   891MB
    node                 latest    1db64f55f800   8 hours ago     936MB
    composer             latest    a2ae9cc28c17   5 days ago      172MB
    nginx                1.19      f6d0b4767a6c   8 days ago      133MB
    php                  7.2-fpm   28f52b60203d   5 weeks ago     398MB
    ```
* タグ付けをすると、レジストリのパスを含むイメージ名が追加される。
    ```
    aws-deploy_nginx                                                latest    58616a6c32d1   13 minutes ago   151MB
    <AWSアカウントID>.dkr.ecr.us-east-1.amazonaws.com/aws-deploy_nginx     latest    58616a6c32d1   13 minutes ago   151MB
    aws-deploy_php_fpm                                              latest    64ea1bcb4963   14 minutes ago   891MB
    <AWSアカウントID>.dkr.ecr.us-east-1.amazonaws.com/aws-deploy_php_fpm   latest    64ea1bcb4963   14 minutes ago   891MB
    node                                                            latest    1db64f55f800   8 hours ago      936MB
    composer                                                        latest    a2ae9cc28c17   5 days ago       172MB
    nginx                                                           1.19      f6d0b4767a6c   8 days ago       133MB
    php                                                              7.2-fpm   28f52b60203d   5 weeks ago      398MB
    ```

## docker container 〜
* `docker container -h`
* 基本的にコンテナの操作において、`docker` の後の `container` は省略しても良い。

## docker (container) run
* `docker run [オプション] [イメージ名:タグ名] [引数]`
* `run` はコンテナの生成と起動を同時に行う。
* dockerイメージがローカルで見つからない場合は、Docker Hubからpullしてくれる。
* `--rm` と `-it`
    * 例: `docker run --rm -it --name phpmyadmin -e PMA_HOST=mysql --network ネットワーク名 -p 8080:80 --mount type=tmpfs,destination=/sessions phpmyadmin/phpmyadmin`
        * `-it` によって標準出力／入力上でphpMyAdminが開始する。Ctrl+Cで終了すると `--rm` によってコンテナが自動的に破棄される。
    * `--rm`: 指定コマンド終了と同時にコンテナを自動的に破棄する。
        * 例: `docker run --rm alpine env`（envを出力して終了後に破棄）
    * `-t` と `-i`
        * `-i`: コンテナ内の標準出力とホスト側の出力をつなげる。
            * 例: `docker run -i ruby:2.4 bash` -> 何もできない。
        * `-t`: ホスト側の入力をコンテナの標準出力につなげる。
            * 例: `docker run -t ruby:2.4 bash` -> 入れるが、何も表示されない。
* `-d`（`--detach`）: バックグラウンドで実行
* `-v=[]`: バインドマウントを作成: `[ホスト側ディレクトリ:]コンテナ側ディレクトリ[:<オプション>]`
* `-e`: 環境変数を渡せる。値を入れない場合はホストに設定されている値をそのまま渡す。

### -vはディレクトリ指定だと上書きになるので注意
* ディレクトリ指定だと、`/var/app` の中身がすべて上書きされる（例えば `/var/app` の中に既に使いたいファイルがある場合は無くなってしまう）。
    ```
    docker run -it --rm -w /var/app -v $(pwd):/var/app gcr.io/distroless/static:debug-nonroot
    ```
    この例の場合、`/var/app` には元々何も入っていないため問題ないが、自分で作成したイメージの `/var/app` にファイルが入っている場合は、実行時に存在しないというエラーになる。
* ファイル指定だと、既存のファイルは上書きされない。
    ```
    docker run -it --rm -w /var/app -v $PWD/firebase.json:/var/app/firebase.json gcr.io/distroless/static:debug-nonroot
    ```
* `/` を指定するとエラーになる。
    ```
    docker run -it --rm -w /var/app -v $PWD:/ gcr.io/distroless/static:debug-nonroot
    # -> invalid specification: destination can't be '/'.
    ```

### -v のマウント先に $pwdや`pwd`を使う場合の挙動差(IME)
* マウント先の記法によって意図しない結果になることがあるため、注意が必要。
    ```
    docker run -it --rm -w /var/app -v `pwd`:`pwd` gcr.io/distroless/static:debug-nonroot
    # コンテナ内の/Users/〜相当のディレクトリが作成されてしまう。マウント先の`pwd`はマウント元の`pwd`と同じ値になる。

    docker run -it --rm -w /var/app -v $pwd:`pwd` gcr.io/distroless/static:debug-nonroot
    # コンテナ内の/:/Users/〜相当にディレクトリが作成されてしまう。

    docker run -it --rm -w /var/app -v $pwd:$pwd gcr.io/distroless/static:debug-nonroot
    # 空白として解釈されるためか、意図しないディレクトリが作成される（$pwdは未定義の変数のため）。
    ```
* ちゃんとディレクトリを指定するのが良い。
    ```
    docker run -it --rm -w /var/app -v `pwd`:/var/app gcr.io/distroless/static:debug-nonroot
    /var/app $ ls
    # 意図通りマウントされている
    ```

## docker (container) cp
* 手元のファイルをコピーできる。
* 例: `docker cp ../setting/ コンテナ名:/tmp/setting`

## docker (container) stop
* `docker container stop {コンテナ名}`
* `docker stop $(docker ps -q)` で全て停止できる。
* デフォルトでは、コンテナのPID=1へSIGTERMを送信し停止させる。10秒経っても停止しない場合はSIGKILLを送信し強制終了する。
    * つまり、すぐに停止しない場合は強制終了になった可能性が高いため注意。

## docker (container) inspect コンテナ名
* コンテナの詳細を確認する。

## docker (container) top
* `docker container top コンテナ名`
* コンテナ内のプロセス一覧を表示する（コンテナ内部で `ps` するのと同じ）。

## docker (container) stats
* リソース使用状況を表示する。`top` コマンドのようなもの。
* `docker stats`
* CTRL + Cで終了。

## docker (container) ls / docker ps
* `docker container ls`（`-a` で停止しているものも全て確認できる）
* または `docker ps`
* `docker container ps` や `docker container list` も同様に使える。

## docker (container) logs
* コンテナのログを表示する。
    * コンテナのPID=1の標準出力（エラー出力も）がlogsになるため、例えばPID=1がbashの場合、そこで操作した内容もlogsコマンドから確認できる。
* `docker container logs コンテナ`
* `-f` で監視、`--tail=10` などでtail表示。
* ログを保存する例
    ```
    mkdir -p nginx/log
    docker run -d -p 80:80 -v $(pwd)/nginx/log:/var/log/nginx --name webserver nginx
    ```

### ログをjqコマンドで見やすくして確認する
* (参考) https://qiita.com/FaithnhMaster/items/03a37642cb48e367c822
    ```
    docker logs コンテナID -f --tail 1 |& grep --line-buffered -io '{.*}' | jq --unbuffered
    ```
* STDOUTとSTDERRの両方を取得するために `|&` を指定。
* jqに渡すためにgrepでjsonのみに絞り込む。
* grepやjqは出力を一旦バッファしてまとめて出力してしまうため、オプションでバッファしないようにしている。

### ログの実体
* Linuxの場合
    * 実体
        * `docker inspect 【コンテナ名】 | grep -i log` で確認できる（あるいは `docker inspect --format='{{.LogPath}}'`）。
        * `/var/lib/docker/container` にある。
    * 削除
        * `echo "" > ログファイル`
* Macの場合
    * HyperKitという仮想化環境でLinuxKitという軽量Linuxを動かしているため、Linuxのように直接確認することはできない。コマンドで確認する方法もあるが、コマンドが長く採用しにくい(IMO)。
        * (参考) https://qiita.com/mingliangli/items/e5e1219a7ac592185d43
        * (参考) https://note.com/w0o0ps/n/n9bc1bcd9fa59
    * 基本的に、直接ログファイルを触る必要はなく、`docker logs` コマンドで十分と思われる(IMO)。

## docker (container) exec
* runningコンテナ内で任意のコマンドを実行する。
* `docker exec コンテナ名 ls`
* 単にコマンドを実行する場合
    * `docker container exec test /bin/ls -l`
    * 手元のファイルを実行したい場合は、`docker container exec -it コンテナ名 bash` などでコンテナ内に入って実行すればよい。
        * (参考) https://qiita.com/shibaHaya/items/ba8f4a7d512e9e062671
* `/bin/bash` などの対話コマンドを実行する場合
    * `docker container exec -it コンテナ名 /bin/bash`
    * Alpine Linuxベースだと `bash` が入っていないことがあるため、その場合は `/bin/sh` を指定する。
    * KubernetesやGoogle Container Engine(GKE)でも同様に `exec -it` オプションで接続可能。
    * リモートホスト上のdockerコンテナについては、dockerにsshdをバンドルするのではなく、ホストOSにSSHログインした上でdocker exec -itという手順を取るのが一般的(IMO)。
    * 接続に成功すると、コンテナ上のプロンプトが表示され、`ps` や `ls` なども使える。作業が終了したら `exit` コマンドで切断する。
    * `exec` で接続した場合は、`exit` してもコンテナは起動したままである（`run` の時にシェルを実行して `exit` した場合は、コンテナは停止する）。
    * rootでログインした状態でコマンドプロンプトが表示される。`-u` オプションで実行ユーザーを指定することも可能。
    * コンテナの中でbashプロセスを新規に立ち上げ、それを操作している。PIDは新たに採番される（PID=1につながるわけではない）。
    * `-i` は `Keep STDIN open even if not attached`（標準入力を開き続ける）、`-t` は `Allocate a pseudo-TTY`（疑似ttyを割り当てる）。`-it` をつけることで、手元の環境からdocker内へ入力できるようになる。

## docker volume 〜
* `docker volume ls`: ボリュームを確認する。
* `docker volume prune`: ボリュームを全て削除する（ボリュームはコンテナを削除しても、削除されずに残る。コンテナが存在する場合は紐づいているボリュームは削除されない）。
* 詳細は[ボリューム](./docker_volume.md)を参照。

## 削除系
* コンテナ
    ```
    docker container rm webserver
    docker container prune  # 停止中のコンテナをすべて削除
    docker rm $(docker ps -q -a)  # -q: only show image ids
    ```
* イメージ
    ```
    docker rmi イメージ名
    docker rmi -f $(docker images -aq)  # -f: 強制削除
    docker rmi $(docker images --filter "dangling=true" -q)  # <none>:<none> のimageを削除
    ```
    特定の名前のimageだけ削除する。
    ```
    docker rmi `docker images --format "{{.Repository}} {{.ID}}" | awk '$1 ~ /aaaaa/ {print $2}'`
    ```
* `docker system prune`
    * イメージ、コンテナ、ネットワークをpruneする。
    * Docker 17.06.0以下のバージョンではボリュームもpruneされていたが、Docker 17.06.1以降では `docker system prune` でボリュームも削除するには `--volumes` フラグが必要。
* 全てを消すやり方（冪等性を担保するために全部消す方法）
    ```
    docker stop $(docker ps -q) ; docker system prune --volumes
    ```
    * image削除に失敗したら、`docker images -aq` して一覧から個別に `docker rmi -f イメージ名` で消していく。
* 削除後の確認
    * `docker ps -a` -> 何も表示されない
    * `docker images -a` -> 何も表示されない
    * `docker network ls` -> デフォルトのbridge, host, noneのみ
    * `docker volume ls` -> 何も表示されない

## docker images
* `docker images`

## docker info
* (参考) https://qiita.com/Shigai/items/bf532649fd9ccf54310b
* `docker info --format '{{.LoggingDriver}}'`
    ```
    json-file
    ```
