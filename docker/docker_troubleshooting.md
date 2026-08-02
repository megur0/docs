---
title: "トラブルシューティング・注意点 - Docker"
updated: 2026-08-02
---

[TOP(About this memo))](../README.md) > [一覧(Docker)](./README.md) > トラブルシューティング・注意点


## ベストプラクティス（TODO）
* (参考) https://qiita.com/takiguchi-yu/items/3202f7bf5620f47f9dee
* rootユーザーは使わないようにする。

## よくあるミス: ポート番号の内部/外部の混同
* 例えばDBのポートを `5433:5432` と指定していた場合、外部からは `localhost:5433` で接続するが、コンテナ間の内部通信では `db:5432` となる。`db:5433` で接続しても繋がらないので注意。

## overlay2まわりのエラー
* docker-composeの再起動時などに「Docker driver "overlay2" failed to remove root filesystem」というエラーが発生することがある(IME)。
    * (参考) https://stackoverflow.com/questions/70971507/docker-driver-overlay2-failed-to-remove-root-filesystem-unlinkat-device-or
    * 対応
        * `grep (出力された/var/lib/docker/overlay2/xxxxx/mergedのxxxxxの部分) /proc/*/mountinfo` でプロセスを特定する。
        * `ps -ef | grep "見つかったプロセス"` で該当プロセスを確認する。
        * 該当プロセスをkillするか、再起動する。
    * (IME) ホスト側の時刻同期デーモン（chronyなど）が影響しているケースがあった。同様の事象が起きた場合は `sudo systemctl stop chronyd` した上で再度コンテナを起動し、`sudo systemctl start chronyd` すると解消することがある。

## マウントしたフォルダへのアクセス権限
* (IME) ホストからマウントしたフォルダに対して、コンテナ内のプロセス（Webサーバーのプロセスなど）からアクセスできるようにする必要がある場合、権限の設定に迷うことがあった。
* 簡易的な対応として `chmod 777` で権限を緩めることもできるが、根本的にはホストとコンテナでUID/GIDを揃える対応が望ましい（[ボリューム](./docker_volume.md)のファイル権限の項を参照）。

## Docker Desktop（Mac）のメモリ設定変更で起動しなくなった
* (IME) Docker Desktopに割り当てるメモリ量を減らして運用していたところ、コンテナが突然落ち、Docker DesktopのダッシュボードでStopping状態のまま進まなくなったことがあった。
* 強制終了した後、再度立ち上げようとするとfatal errorになったため、Docker公式フォーラムの手順を参考に復旧した。
    * (参考) https://forums.docker.com/t/cannot-get-docker-working-in-macbook-pro-m1/120810/3
    * 復旧手順の中でメタファイル関連の権限エラーが出たが、無視して問題なかった。
* この際、コンテナのイメージなどは消失した。
* 復旧後に再度コンテナを立ち上げようとしたところ、Dockerのゾンビプロセスがコンテナで使っていたポート（Postgresの5432/5433番など）を占有しており、`kill -9 プロセスID` で強制終了する必要があった（`kill プロセスID` では終了しなかった）。
* (IME) 結論として、Docker Desktopに割り当てるメモリ量はむやみに変更しないほうがよさそうだ。
