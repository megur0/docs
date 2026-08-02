---
title: "Docker Hub - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > Docker Hub


## Docker Hub
* あらかじめよく使われる便利なイメージが登録されたレジストリ。

## Docker Hubのイメージを改造したい場合（php-fpmの例）
* (参考) https://qiita.com/shim-hiko/items/653059fab63af962a21f
* 改造先で `CMD ["test.sh"]` のように書くと、元々入っているphp-fpmコマンドの代わりにtest.shを実行してくれる。あとはtest.shの中でphp-fpmコマンドを実行すれば問題なく動作する。

### CMDもENTRYPOINTも一度しか呼べず、後から呼ぶと上書きされる
* (参考) https://github.com/docker-library/php/tree/74175669f4162058e1fb0d2b0cf342e35f9c0804/7.3/buster/fpm

### CMDとENTRYPOINTを一緒に使った時の挙動
* ENTRYPOINTとCMDの両方を書くと、上書きできないものと上書きできるものの両方を実行することになり、両者を連結して実行してくれる。
* (参考) https://peccu.hatenablog.com/entry/2015/06/20/000000

### CMDを上書きする例
* 以下のように書いてある場合、`docker run imagename example.com` を実行すると `/bin/ping -c 3 example.com` で実行される。
    ```dockerfile
    ENTRYPOINT ["/bin/ping", "-c", "3"]
    CMD ["localhost"]
    ```
* nginxイメージのCMDを `nginx -g "daemon off;"` で上書きして実行させる例。
    ```
    docker run -d -p 80:80 nginx nginx -g "daemon off;"
    ```

## Docker Hubのイメージ
* バージョン指定が「1」のように曖昧だと、イメージをpullするタイミングによってマイナーバージョンが変わってしまう。
* 同じバージョン表記でも、実際は更新がされていることがある。
    * (参考) https://qiita.com/ozota/items/93f9b7568dc5b262996e
* つまり、イメージを削除して再度pullする場合は、前もって内容を確認しておく必要がある。

### Docker Hubから持ってきたイメージを見てみる
* `docker images` でイメージ一覧を見る。
* `docker inspect nginx:1.19` のようにすると内容を確認できる。
    * アーキテクチャは例えば `"Architecture": "amd64"` のように書いてある。
    * Docker Hubに置いてあるイメージはマルチアーキテクチャに対応していることが多く、pull時に自動的にホスト環境のアーキテクチャに応じたイメージを持ってきてくれる。
        * (参考) https://dev.classmethod.jp/articles/docker-multi-architecture-image/

### pullするときに指定できるタグ
* Docker Hubのサイトで確認できる。対応しているアーキテクチャも書いてある。
    * 例: https://hub.docker.com/_/nginx?tab=tags&page=3
* nginxなど、GitHubでコードを確認できるイメージもある。
    * デフォルトで持ってくるのはmainline/buster系が中心のようだ(?)。
    * mainline、stableという系統があり、さらにalpineやbusterといったベースOSのバリエーションがある。DockerfileにはNGINX_VERSION、NJS_VERSION、PKG_RELEASEといった環境変数があり、これらと `docker inspect` の内容を照らし合わせると、指定したタグでどのファイルを持ってきているか概ね把握できる。
    * (参考) https://github.com/nginxinc/docker-nginx/tags?after=1.19.4

### pullの細かい指定方法
* (参考) https://dev.classmethod.jp/articles/docker-multi-architecture-image/
* `docker pull <アーキテクチャ名>/<サブリポジトリ名>:<タグ名>`
    * Dockerを実行している環境によらず、任意のアーキテクチャに対応したイメージをpullできる。
* `docker pull <リポジトリ名>:<タグ名>@sha256:<DIGEST>`
    * DIGESTはDocker HubのWebサイトから確認できる。

## イメージファイルの中身を見る
* `docker history --no-trunc nginx:1.19 > output.txt`
    * (参考) https://engineer.namake-mono.xyz/development/381/
* `missing` と表示されている箇所は「レイヤーの実体がローカルに存在しない（親イメージ側のレイヤー）」ことを示す(?)。
    ```
    IMAGE       CREATED       CREATED BY
    sha256:〜〜〜   3 weeks ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon off;"]
    ...
    <missing>   3 weeks ago   /bin/sh -c #(nop)  CMD ["bash"]
    <missing>   3 weeks ago   /bin/sh -c #(nop) ADD file:7362e0e50f30ff45463ea38bb265cb8f6b7cd422eb2d09de7384efa0b59614be in /
    ```

## alpineは基本的に避けたほうがよい場合がある
* (参考) https://engineering.nifty.co.jp/blog/26586
* (参考) https://blog.inductor.me/entry/alpine-not-recommended
* (参考) https://zenn.dev/jrsyo/articles/e42de409e62f5d
* Alpine Linuxでは標準Cライブラリとしてglibcの代わりにmusl libcが使われており、一般的な互換性が不足しがち。
* Ruby、Python、Node.jsなどでネイティブモジュールをバンドルしているアプリケーションの場合、パフォーマンスの劣化や互換性の問題が発生する場合がある。
    * なお、Goではデフォルトでlibcに依存しているが、`CGO_ENABLED=0` とすることでlibc非依存にできる。
