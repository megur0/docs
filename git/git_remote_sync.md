---
title: "push・pull・fetch"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(Git)](./README.md) > push・pull・fetch


# push
* 上流ブランチを設定しておくため、オプション`-u`をつけておいた方がよい（上流ブランチについては[branch_merge](./git_branch_merge.md)を参照）。

## pushの取り消し
```
git push -f origin HEAD^:<ブランチ名>
```
* このコマンドは整合性の面で結構危ないため、過去に遡って行うのは避けた方がよいかもしれない(IMO)。
* 注意したいのは、ローカルのcommitは残るため、ローカルも合わせて戻すにはpushを取り消した後に以下も実行しておく。
    ```
    git reset origin/xxx
    ```
* 履歴を残す場合はrevertを使う方がよい（基本的にこちらを使う）。

## `push -f`は共同作業ではあまり使わない方がよいかもしれない
* (参考) https://qiita.com/hnw/items/5ac4416c72dd567b263f
* 他人もpullしている場合だと、下のような状態になってしまうことがある。
    ```
    * - * - * - * - * - * - *
          \                 ↑
            *               origin/foobar
            ↑
            remotes/origin/foobar
    ```
* 自分しか使っていないブランチならOKだが、developなどの共有ブランチでこれをやるのはNG。
* 上記のような状態になると、`git fetch`や`git pull`で下記のようなメッセージが出る。
    ```
    Your branch and 'origin/foobar' have diverged,
    and have 5 and 1 different commit each, respectively.
    ```
* 解決手段は、`git reset --hard origin/foobar`でリモート側に合わせる。


# pull
## 基本
* リモートリポジトリにpushされた変更をローカルに反映する。
    ```
    git pull <リモートレポジトリ名> <ブランチ名>
    ```
    は以下と同じ。
    ```
    git fetch <リモートレポジトリ名> <ブランチ名>
    git merge <リモートレポジトリ名>/<ブランチ名>   # git merge FETCH_HEAD でもOK
    ```
* 引数なしで`git pull`を実行すると、今チェックアウトしているブランチの上流ブランチだけpullする。つまり以下。
    ```
    git pull <今いるブランチの上流ブランチ>
    ```
    これは以下と同じ意味。
    ```
    git fetch <今いるブランチの上流ブランチ>
    git merge <今いるブランチの上流ブランチのリモート追跡ブランチ>
    ```
    * 例えば、今のブランチが`feature-xxx`のときに`git pull origin test`とすると、`feature-xxx`に`test`がマージされる（マージのされ方は[branch_merge](./git_branch_merge.md)参照）。
* 全てのリモート追跡ブランチを更新したい（ローカルにないものも含む）場合は`git fetch`する。

## 衝突時・エラー発生時
* pullして衝突した場合（`Your local changes to the following〜`と出た場合）
    ```
    git stash -u   # まず作業ファイルを退避する
    git stash list # 確認
    git pull
    git stash pop
    ```
* pullして衝突した場合（オートマージされずコンフリクトになった場合）
    * 目で見て修正し、`git add`してから`git commit`する。
* pullして衝突した場合（強制的にpullする場合）
    ```
    git fetch origin master
    git reset --hard origin/master
    ```
* リモート追跡ブランチはあるが上流ブランチが設定されておらず、`git pull`してもマージされない場合
    ```
    git branch --set-upstream-to=origin/deployment deployment
    ```
    * `git push`で`-u`をつけ忘れた場合などにも使える。設定後、再度`git pull`する。
* `git pull`で`error: cannot lock ref 〜`のようなメッセージが出た場合
    * リモートで削除されたブランチなどをローカル側でも削除する。
        ```
        git remote prune origin
        ```


# fetch
## 引数なし`git fetch`
* 全てのリモート追跡ブランチを更新する（ローカルに存在しないブランチも取得する）。
* `git fetch origin +refs/heads/*:refs/remotes/origin/*`と同じ意味。originの全てのheadsをremotes/origin以下に写すという意味。

## checkoutせずにmergeしたい（その1）→非推奨
* 本質的に`git pull`するのと挙動が違うため、使うとしてもよく理解してから使った方がよい(IMO)。
* (参考) https://stackoverflow.com/questions/36214756/what-does-git-fetch-origin-mastermaster-mean
* 本質的には、ステージング環境ではstagingにマージしてから、ステージング環境で`git pull`するだけで済む気もするが、複数の作業者が同時に作業する場合は現実的ではない。
* `git fetch origin master:master`でチェックアウトせずにfetchとマージができる（fast-forwardのみ。fast-forwardでない場合はエラーになる）。
    * 対象ブランチに既にいる場合もエラーになるので注意。
* 以下のようにやった場合、ソースコードが変わってしまうため、環境によっては（例えばキャッシュを持つフレームワークなどでは）不具合を起こしかねない。
    ```
    git fetch
    git checkout master   # この時点でソースコードが切り替わり、環境に影響を与える可能性がある
    git merge
    ```
* (参考) https://jpcodeqa.com/q/c85eb5377d6bd56171f909bc00f5aba5

## checkoutせずにmergeしたい（その2）→こちらの方がよさそう
```
git branch -D <ブランチ名>
git checkout <ブランチ名>
```
* これなら古い内容が反映されることもない。

## `git fetch <リモートレポジトリ名> <ブランチ名>`の動作
* (参考) http://www-creators.com/archives/2295
* `git fetch`は「リモートブランチの履歴を取り込む」コマンド。ローカルブランチ、インデックスや作業ツリーには変更を加えない。
* `git fetch`だけの場合（引数がない場合）、全てのリモート追跡ブランチのみが更新される(?)。
* これと同時に、fetchされたコミットオブジェクトへの参照が`FETCH_HEAD`に書き込まれる。
    ```
    リモート(origin)：X-X-X-X(test)
    ローカル        ：X(test)-X-X-X(origin/test)   # リモート追跡ブランチが更新される
    ```
* 他のブランチをマージしたい場合
    ```
    git fetch          # 全てのリモートの更新をフェッチする
    git merge origin/test
    ```

## リモート追跡ブランチ（リモートトラッキングブランチ）について
* `git branch -a`で全てのブランチを表示すると、「remotes/origin/feature-a」のようにブランチ名の前にリポジトリ名が表示される。この「remotes/xxxx/xxxx」となっているブランチ名が「リモート追跡ブランチ」である。
    * 出力例
        ```
        * feature-a
        remotes/origin/HEAD -> origin/master
        remotes/origin/master
        remotes/origin/feature-a
        ```
* 「リモート追跡ブランチ」は種類としてはローカルブランチである。しかし、自分がコミットを行う類のブランチではなく、常に対応するリモートブランチからfetchして更新される対象として区別されている。通常のローカルブランチのようにチェックアウトして作業することはできない（`git checkout remotes/origin/feature-a`のようなことはできない）。
* この「リモート追跡ブランチ」はリモートリポジトリにある全てのブランチ分が最初から存在するわけではないため、`git fetch`することで取得する。
* `remotes/origin/HEAD -> origin/master`は、リポジトリのデフォルトブランチを表す。例えば`origin/HEAD`が`origin/master`を指しているとき、`git checkout -b test origin`とすると`git checkout -b test origin/master`と同じ意味になる。デフォルトブランチは`git remote set-head`で変更できる。
* `git push`成功時には（上流ブランチとして設定されていれば）自動的に対応するリモート追跡ブランチの参照が更新される。上流ブランチが設定されていない場合は、`git push -u origin master`などの`-u`オプションで設定できる。

## リモートにあるブランチを更新したいとき
* リモートにあるブランチ（ローカルにないブランチ）を更新したいとき、ついローカルブランチのようにチェックアウトして編集したくなるかもしれないが、Gitの仕様上、作業ブランチをリモートブランチに切り替えて直接更新することはできない。ローカル環境からリモートブランチを更新する唯一の手段は「ローカルブランチの履歴をリモートブランチにpushする」ことのみとなる。通常は以下のように、fetchしてから作業する。
    ```
    # リモートのcool-featureブランチに対応する追跡ブランチを作成
    git fetch origin cool-feature
    ```
    * この時点では、まだ作業用のブランチは作られておらず、リモート追跡ブランチ「origin/cool-feature」が存在するのみ。
    ```
    # 自動的に同名のリモート追跡ブランチを起点にして、ローカルブランチを新規生成しチェックアウト
    git checkout cool-feature
    ```
    * まだローカルブランチが存在しないため`-b`が必要な気もするが、fetchでリモート追跡ブランチ「origin/cool-feature」が作成されているため、自動的にそれを起点にしてローカルブランチの作成をしてくれる親切設計になっている。
    ```
    # ローカルブランチで変更を行ったcool-featureの状態を、対応する上流ブランチにpushする
    git push
    ```
    * 上流ブランチが設定されているため、「`git push origin cool-feature`」とする必要はない。
    * この方法を使うと、同時に`cool-feature`ブランチの上流ブランチ（upstream branch）として「origin/cool-feature」が設定される。
    * `git checkout -b <branch> --track <remote>/<branch>`と同じ動作。
