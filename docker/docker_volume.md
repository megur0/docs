---
title: "ボリューム - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > ボリューム


## volume
* (参考) https://www.slideshare.net/zembutsu/docker-compose-guidebook （98ページあたり）
* volumeとは、コンテナのライフサイクルが終了した後でもデータを保管しておけるデータ領域。
* volumeにも種類がある。

### bind mount
* ホスト側のディレクトリをvolumeとしてコンテナ内にマウントできる（bindする、という）。
    ```
    docker container run \
      --rm \
      -v $(pwd):/mnt \
      -it ubuntu:18.04 \
      bash
    ```
    * カレントディレクトリが `/mnt` にマウントされる。例えばカレントディレクトリに `greet.txt` を作成すると、`/mnt/greet.txt` がコンテナ上でも作成される。

### volume mount
* 公式でもこのやり方が推奨されている。
    * (参考) https://blog.amedama.jp/entry/docker-mount-volume
* `docker volume create example-volume`
* `docker volume ls` で確認するとボリュームがある。
* volume mountするときは、bind mountと同じように `-v` オプションが使える。bind mountでは左辺にDockerホストのディレクトリを指定したのに対し、こちらではボリュームの名前を指定する。
    ```
    docker container run \
      --rm \
      -v example-volume:/mnt \
      -it ubuntu:18.04 \
      bash
    ```
* あるいは `--mount` オプションを使ってもよい。この場合は `type` に `bind` ではなく `volume` を指定した上で、`source` にボリュームの名前を指定する。
    ```
    docker container run \
      --rm \
      --mount type=volume,source=example-volume,destination=/mnt \
      -it ubuntu:18.04 \
      bash
    ```
* `docker ps` で「CONTAINER ID」を確認し、`docker inspect コンテナID | grep ボリューム名` とすると確認できる。実体は `/var/lib/docker/volumes` あたり（`sudo ls /var/lib/docker/volumes/ボリューム名/_data`）。

### その他
* tmpfs mount
* delegate、cache（TODO）
    * (参考) https://qiita.com/ysKey2/items/346c429ac8dfa0aed892

## Dockerのファイル権限
* (参考) https://qiita.com/yohm/items/047b2e68d008ebb0f001
* (参考) https://create-it-myself.com/know-how/set-host-userid-groupid-to-docker-container/
* Linuxで、Dockerにファイルをホストからマウントすると、コンテナ内で作成されるファイルはデフォルトでownerがrootになってしまう。この問題はDocker for Macでは発生しない（コンテナ内からは所有者はrootだが、ホストから見ると自ユーザーがownerとして扱われる）。
* 対策
    * `docker run` の際に `-u` で指定する方法
        * `docker run -it -u \`id -u $USER\` debian:jessie /bin/bash`
        * gid（group id）が変わっていない、かつ `/etc/passwd` の情報とuidが一貫していないという問題が残る。
            * ファイルを使うだけなら問題ないが、いくつかのアプリケーションが `/etc/passwd` を参照することがあり、問題が起きる場合もある。
        * `docker run -it -u \`id -u\`:\`id -g\` debian:jessie /bin/bash` とすることで解消できる場合がある(?)。
    * ホストのユーザーと同じuid・gidで `useradd`・`groupmod` する方法（こちらのほうが確実(IMO)）
        * あらかじめDockerfile等で作成しておき、`RUN` の際にそのuid:gidを指定するか、`command` でシェルスクリプト（useradd、groupmodした上でそのユーザーにsuする）を実行する。
        * docker-composeであれば、例えば `user: 1000:1000` のように設定できる。
