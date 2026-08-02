---
title: "基本概念 - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > 基本概念


## Dockerのコンセプト
* (参考) https://knowledge.sakura.ad.jp/23899/
* アプリケーションの開発・移動・実行を簡単にするためのソフトウェア。

## 仮想化の種類

### ホスト型: ゲストOSがホストOSを経由してハードウェアにアクセス
* OSにベースとなる仮想化ソフトウェアを導入し、その上で仮想マシンを稼働させる方式。
    * ホストOS: ハードウェア上に展開するOS
    * ゲストOS: 仮想化ソフトウェア上で稼働するOS
* ゲストOSがハードウェアにアクセスするにはホストOSを経由する必要があり、その分の処理時間・処理負荷がかかる。
* 仮想化ソフトウェアをインストールするだけで構築可能。

### ハイパーバイザー型: ゲストOSが直接ハードウェアにアクセス
* OSに対してではなく、物理サーバーへハイパーバイザー（仮想化ソフトウェア）をインストールする方式。
    * ホストOSが不要。
    * ハイパーバイザーがハードウェアを直接制御できるため、処理速度の低下を抑えられる。
* 企業で仮想化を行う場合、比較的一般的な方式(IMO)。
* 専用の物理サーバーを用意する必要がある。

## コンテナと仮想マシンの違い
* (参考) https://www.rworks.jp/system/system-column/sys-entry/21776/
* (参考) https://qiita.com/kirikunix/items/33414240b4cacee362da
* コンテナは、ゲストOSそのものを立ち上げているのではなく、ゲストOSの「動作を再現」しているだけ。
    * コンテナエンジン（仮想化ソフトウェア）をインストールすることで、コンテナ（仮想的なユーザー空間）はOSのカーネルを利用して動作する。
    * ホストOS（Linux）のABI（Application Binary Interface。アプリケーションとLinuxカーネルの間のインターフェース）を使って動作を再現しており、Linuxのバージョンが違っても互換性に配慮されている（ファイルシステムやライブラリなども再現される）。
    * これにより処理の軽量化を実現している。
    * ただし、ホストOS(Linux)が持っていないABI（最新のLinuxカーネルの機能など）は再現できない。
* MacやWindowsでは、間にLinuxを挟んで動作させている（ホストOSがLinuxである必要があるため）。その分オーバーヘッドが生じる(IMO)。

## フォアグラウンド実行とPID1
* Dockerコンテナは、フォアグラウンドで動いているプロセスがなければ停止してしまう。

### PID1のプロセス
* (参考) https://ngzm.hateblo.jp/entry/2017/08/22/185224
* (参考) https://casualdevelopers.com/tech-tips/how-to-fix-the-pid-1-problem-for-dockerized-nodejs-app/
* コンテナ内でPID 1として実行しているプロセスは、Linuxが特別に扱う。デフォルトの動作では、あらゆるシグナルを無視する。そのため、プロセス側でSIGINTやSIGTERMを受けて停止するように実装しない限り、プロセスは停止できない。
    ```
    docker exec -it コンテナ名 /bin/bash
    kill 1  # -> 停止しない
    ```
* `docker stop` を実行すると、Dockerはコンテナのルートプロセス（PID=1）にSIGTERMを送る。
* SIGTERM送信後、Dockerはルートプロセスが終了するまでデフォルトで10秒間待つ。10秒経過しても終了しない場合は、SIGKILLを送って強制終了する。

### PID1・子プロセス・シグナル伝搬の注意点
* (参考) https://cloud.google.com/blog/ja/products/application-development/graceful-shutdowns-cloud-run-deep-dive
* (参考) https://qiita.com/ukinau/items/410f56b6d777ad1e4e90
* (参考) https://qiita.com/kazurego7/items/57f5fb80b4783b7633a1
* 例えば、CMD（あるいはdocker-compose.ymlの `command`）を `sh -c "nginx -g 'daemon off;'"` のように実行すると、シェルは子プロセスにシグナルを伝搬しないため、Ctrl+Cなどで SIGTERM を送っても子プロセスが終了処理をできず、結果的にSIGKILLされて終了してしまう。
* また、`CMD "nginx -g 'daemon off;'"` はシェル経由（`sh -c "nginx -g 'daemon off;'"`）で実行される点にも注意。
    * `CMD ["nginx", "-g", "daemon off;"]` の形式であればシェルを経由しない。
