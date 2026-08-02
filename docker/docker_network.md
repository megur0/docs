---
title: "ネットワーク - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > ネットワーク


## ネットワーク
* (参考) https://knowledge.sakura.ad.jp/23899/
* (参考) https://qiita.com/TsutomuNakamura/items/ed046ee21caca4a2ffd9
* 通常のLinuxホスト上では、複数のプロセスがポートを重複して開くことはできない。
* Dockerでは、
    * PIDなどの名前空間が隔離（isolate）され、お互いのプロセスが見えない状態で動作する。
    * コンテナごとにネットワークも隔離されるため、コンテナ内で各々のプロセスがポートをリッスンしていたとしても（ホスト側で使用するポート番号が重複しなければ）お互いに影響を与えず起動し続けられる。
* Dockerにはコンテナ間で通信したり処理の負荷分散したりするため、ネットワーク内部で利用可能な「コンテナ名」「IPアドレス」の名前解決（サービス・ディスカバリ）を行う仕組みが標準で搭載されている。
    * そのため、コンテナ間の通信はコンテナのIPアドレスを指定するのではなく、通信先をコンテナ名で指定できる（スケールできる環境においてもIPアドレスを意識する必要がなくなる。コンテナ間の通信や負荷分散が容易になるのは、コンテナ導入の大きなメリット）。

## 3つのネットワーク
* Dockerをインストールすると、3つのDocker Networkが標準で作成される。`docker network ls` コマンドで確認できる。
    ```
    NETWORK ID          NAME                DRIVER
    10aa51e2d993        bridge              bridge
    202b87d7b8a4        none                null
    92316fc5522f        host                host
    ```
* noneネットワーク
    * コンテナにネットワーク・インターフェースを持たせない。そのため、コンテナは内外の通信ができない。
* hostネットワーク
    * コンテナがホスト側のネットワーク・インターフェースを共有する。ネットワークが隔離されないため、コンテナ内でプログラムがポートをリッスンすると、そのままインターネット側と通信可能になる。
    * 指定する場合の例
        ```
        docker run -itd --net host --name hostnginx nginx:alpine
        ```
        * 通常はホスト外からの通信をするには `-p` オプションの指定が必須だが、hostネットワークを使う場合は、コンテナとして実行しているnginxがリッスンしているポート80が、特に何の制限もなくホスト外からも通信できる。
* bridge
    * Linux bridge機能を使った、Linux上に別ネットワークを構成する方式。WindowsやMacのDocker環境の場合は間にLinuxを動かし、そのLinuxのLinux bridge機能を使う。
    * 「ブリッジ」とは一般的なネットワーク機能であり、別々のネットワーク・セグメント間で通信できるようにする働きがある。Dockerにおけるブリッジ・ネットワークも似たような概念で、同じブリッジ・ネットワーク上に接続しているコンテナはお互いに通信できる。
    * Dockerのコンテナ作成時に `--net` オプションを特に指定しない場合、自動的にこれが選択される。
        * ブリッジネットワークの状態をコマンドで確認する例: https://knowledge.sakura.ad.jp/23899/
    * 別のブリッジネットワーク同士だと通信できない。
    * 何も考えずに `docker run` コマンドを実行すると、通常は「bridge0」という名前のブリッジ・ネットワークに接続する。
        * デフォルトの「bridge0」という名称のブリッジ・ネットワークでは名前解決機能がないため、IPアドレスでの疎通は可能だが、コンテナ名での通信はできない。コンテナ間の通信でサービス・ディスカバリを有効化するには、自分でブリッジ・ネットワークを作成する必要がある（docker-composeを使っていれば、何も指定しなくてもデフォルトでネットワークを作成してくれる）。
        * (?) docker on Macだと「bridge0」に相当するインターフェースが見当たらないことがある。
* overlay network
    * 異なるDockerホスト上に存在するコンテナに対して、同じネットワークに存在するコンテナとして透過的にアクセスできる。
    * 同じネットワーク上であれば、例えば`geth`というノードに対して `http://geth:8545` のような形でアクセスできる。
        * (参考) https://docs.docker.jp/compose/networking.html

## docker composeのネットワーク機能
* (参考) https://docs.docker.jp/compose/networking.html
* デフォルトで作成されるネットワークの例
    * `myapp` というフォルダにあるdocker-compose.ymlで `up` すると「myapp_default」というネットワークが生成される。それぞれのコンテナが `myapp_default` に参加する。
* `external: true`
    * 指定すると、`name` で指定した既存のネットワークに接続する。
* `internal: true`
    * 外部とのアクセスができないネットワークになる（デフォルトは`false`）。
        * (参考) https://knowledge.sakura.ad.jp/4786/
        * (参考) https://www.timedia.co.jp/tech/20220628-tech/
        * (?) 参考記事によってtrue/falseの解釈の説明が異なって見えることがあるため、実際の挙動は手元で確認したほうがよい。

## ネットワークの確認
* `docker network inspect ネットワーク名`
    * 指定したネットワークの情報を確認する。
* 作成したネットワークがどうなっているか確認する例
    ```
    docker network create mybridge
    docker run -itd --net mybridge --name alpine3 alpine /bin/sh
    docker network inspect mybridge
    ```
    * `ping 172.19.0.2` や `ping db` で疎通を確認し、他のブリッジネットワークには接続できないことを確認する。
    * `docker stop コンテナID`
    * `docker container prune`

## --network hostはMacだと機能しない
* hostネットワークドライバーはLinuxホスト上でのみ動作する。Docker Desktop for Mac、Docker Desktop for Windows、Docker EE for Windows Serverではサポートされていない。
    * (参考) https://matsuand.github.io/docs.docker.jp.onthefly/network/network-tutorial-host/
* 単にホスト上のポートで公開されているコンテナにアクセスすることはできた(IME)。
    * 例えばローカルで `5432:5432` としてポートをpublishしているPostgresのコンテナに対して、`--network host` で立ち上げた別のコンテナからアクセスすることはできた。
* しかし、`--network host` として立ち上げたサーバーに対して、ホストマシンからアクセスすることはできなかった(IME)。
    * 例: 下記でnginxを立ち上げても `localhost:80` でアクセスできない。
        ```
        docker run --rm --network host --name my_nginx nginx
        ```

## host.docker.internal
* Docker内から見たホストのDNS。これを使えば、ホスト側で立ち上げたサーバーなどにアクセス可能。
* 例: `http://host.docker.internal:8080/〜`

## ポートマッピングについて

### dockerはデフォルトで自身のマシン（0.0.0.0:xxxx）へのアクセスを許可してしまう
* 公式にも注意事項として記載がある。
    * (参考) https://docs.docker.com/network/
    * (参考) https://jun-networks.hatenablog.com/entry/2023/07/03/190000
* つまりデフォルトでは、同じLAN内の端末からアクセスができてしまう（公共のWi-Fiなどに繋ぐと、アクセスされてしまう可能性がある）。
* 例えば、
    * `docker container run -p 8080:80 nginx` の場合、`0.0.0.0:8080 -> 0.0.0.0:80` という形でバインドされる(IMO)。
    * `docker container run -p 127.0.0.1:8080:8080:80 nginx` のようにすると、自分のマシンからのみアクセスできる。
* `docker container port コンテナ名` で確認すると `8080/tcp -> 0.0.0.0:8080` としか表示されない。ちゃんと `0.0.0.0:8080/tcp -> 0.0.0.0:8080` のように出してくれたほうが親切だと思うのだが(IMO)。
* `lsof -i -P` で確認するとバインド状況が分かる。
    ```
    com.docke  2349  <ユーザー名>  163u  IPv6 0x8c33e5bd6b030031      0t0  TCP *:8080 (LISTEN)
    ```

### ポートマッピングはコンテナ内の0.0.0.0をマッピングする
* したがって、コンテナ内のサーバーを立てる場合は、`127.0.0.1` ではなく `0.0.0.0` にする必要がある。

## 他
* dockerによるnetworkとdocker-composeによるnetwork
    * (参考) https://qiita.com/KEINOS/items/42aae92d00675c8b0b78
