---
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(Git)](./README.md) > ブランチ・マージ


# branch
## 覚えておきたいこと
* `git branch -a`：`remotes/origin/〜`と表示されるのがリモート追跡ブランチ。
* `git branch -vv`：`[origin/develop: behind 2]`のように表示されるのが上流ブランチ。
* `git status`でも現在のブランチと上流ブランチがわかる。

## 基本
* `git branch`：すべてのブランチを表示（`*`がついているのが今のブランチ）。
* `git branch <newbranch>`：新しいブランチを作成する。
* `git checkout -b <newbranch>`：新しいブランチを作成し、すぐにそのブランチに切り替える。`git branch <newbranch>`してから`git checkout <newbranch>`するのと同義。
    * (参考) https://code-examples.net/ja/q/79e1e7 （`git branch`と`git checkout -b`の違い）
* 作成したブランチをpushする
    ```
    git add .
    git commit -m "****"
    git push -u origin <newbranch>   # -uをつけないと上流ブランチに設定されない
    ```
* ブランチの削除（例：`develop`から派生した`feature-xxx`を削除する場合）
    ```
    git checkout develop
    git branch -d feature-xxx
    ```
    * (参考) https://qiita.com/shuntaro_tamura/items/6c8bf792087fe5dc5103

## 既にpush済みのブランチの名前変更
* 手順
    1. ローカルを変更して変更後の名前をpush
    2. 変更前の名前をリモートから削除
    ```
    git branch -m <変更前> <変更後>
    git push -u origin <変更後>
    git push origin :<変更前>
    ```
* (参考) https://qiita.com/shungo_m/items/4218e70751375b4bfeec


# 上流ブランチの確認・設定方法
* 注意：リモート追跡ブランチが存在するからといって、上流ブランチが設定されているとは限らない。
* 上流ブランチの設定確認
    ```
    git branch -vv
    ```
    * 結果例：masterの上流ブランチはorigin/master（リモート追跡ブランチそのものではないので混同しないよう注意）。
        ```
        master       b104f69 [origin/master: ahead 1] xxxx xxxxx xxxxxxxxxxxx xxx.
        cool-feature b104f69 [origin/cool-feature] xxx xxxxxxxx xxxxxxxx xxxx xxxxx.
        ```
* 上流ブランチの設定方法
    1. 明示的に上流ブランチを設定（以下の2つは同義）
        ```
        git branch <ローカルブランチ名> -u <リモートブランチ名>
        git branch <ローカルブランチ名> --set-upstream-to=<リモートブランチ名>
        ```
        * ローカルブランチ名を省略すると、自動的に現在のブランチが対象になる。
        * 例：`git branch -vv`で確認するとローカルブランチ`schedule`に上流ブランチが設定されていない場合、`git branch -u origin/schedule`を実行すると、`git branch -vv`で`schedule`に上流ブランチが設定されていることを確認できる。
    2. `git branch`でリモート追跡ブランチを起点にブランチを作成する
        * すでに`origin/cool-feature`が存在する状態で
            ```
            git branch origin/cool-feature
            ```
            結果：
            ```
            Branch remote set up to track remote branch cool-feature from origin.
            ```
        * これはGitの親切設計で、新規ブランチ作成時に起点としてリモート追跡ブランチを明示すると、自動的に該当のリモートブランチを追跡ブランチとして設定してくれる。
    3. `git checkout`にリモートブランチ名を指定する（実際にはこのケースが多いかもしれない(?)）
        * すでにリモート追跡ブランチ`origin/cool-feature`が存在する状態で
            ```
            git checkout cool-feature
            ```
            結果：
            ```
            Branch cool-feature set up to track remote branch cool-feature from origin.
            Switched to a new branch 'cool-feature'
            ```
    4. push時に同時に上流ブランチとして設定する
        * push時に「`-u`（`--set-upstream`）」オプションを付ければ、自動的にpushするブランチとpush先のリポジトリ・ブランチが上流リポジトリ・上流ブランチとして設定される。フィーチャーブランチを初めてリモートにpushするときに便利なので、よく利用する。
        * push先を上流ブランチとして設定（以下の2つは同義）
            ```
            git push -u origin cool-feature
            git push --set-upstream origin cool-feature
            ```
    5. `.git/config`に直接書き込む（ほぼ使わない）
        * `cool-feature`の上流ブランチを`origin/cool-feature`に設定
            ```
            git config branch.cool-feature.remote origin
            git config branch.cool-feature.merge refs/heads/cool-feature
            ```


# merge
* (参考) http://www-creators.com/archives/4996

## `--no-ff`をつけておく
* なぜつけるか
    * fast-forwardのmergeの場合、mergeのコミットが発行されないため、masterがその後更新されていくとtopicブランチで行われた作業を参照するのが面倒になる。
    * mergeの取り消しを行いたい場合にも、fast-forwardだと面倒になる。
* 作業ブランチとして特定の意味を持たないブランチの場合は、fast-forwardのmerge（オプションなし）でもよい。
* 逆に、たとえばstagingへの反映をdevelop-xxxに反映するときなどは、GitHubなどのマージ機能を使うと`--no-ff`が入るはずで、マージコミットが増えていくのを避けたい場合はローカルで`git merge origin/staging`してからpushすることもある(IME)。
* (参考) https://qiita.com/nog/items/c79469afbf3e632f10a1

## mergeの動作
* 現在チェックアウトしているブランチに対して、別のコミットが持っている変更内容をマージ（統合）するためのコマンド。
* 「コミットが持つ変更」とは、正確には「現在のブランチの履歴と、指定コミットの履歴とが分岐した地点から指定コミットまでに及ぼされた変更内容」。詳細動作は以下の通り。
    * `MERGE_HEAD`参照が、指定したブランチの先頭を指すように設定される。
    * HEADの参照はそのまま。
    * マージ処理を開始し、問題なく成功したファイルの状態がインデックスと作業ツリーにそれぞれ反映される。
    * 競合が発生したファイルは、インデックスに3つのバージョンとして保存される（履歴が分岐する前の共通の状態、HEADの状態、MERGE_HEADの状態）。作業ツリーは`<<<`、`===`、`>>>`の記号を使って、開発者に競合箇所を明示するように更新される。マージに関係ないファイルが変更されることはない。
    * 1箇所以上のコンフリクトが発生すれば処理が中断し、コンフリクトの解決は開発者に委ねられる。

## 衝突
* 同じファイルでも修正箇所が違う場合は衝突しない。

## マージ方法
* 1. 異なるブランチをマージする場合
    ```
    // ブランチの状態、現在masterブランチをチェックアウトしている
      o---o :topic
     /
    -o----o :master <- HEAD

    // git merge topic 実行後
       o---o :topic
     /        \
    -o----o----o :master <- HEAD
    ```
* 2. Fast-forward（早送り）マージ
    ```
    // マージしたいブランチ（topic）が現在のブランチ（master）の直接的に先に進んでいる場合
    // ブランチの状態、現在masterブランチをチェックアウトしている
       o---o :topic
      /
    -o :master <- HEAD

    // git merge topic 実行後
    // この場合、変更が存在しないのでマージにあたってコミットが作られない（fast-forwardマージ）
    // つまり、参照だけ移動される
        o---o    :topic, master <- HEAD
      /
    -o
    ```
    * `--no-ff`オプションを加えることで、強制的に新しいコミットを生成することも可能。
    * 基本的にはマージの履歴は残した方がよいため、付けておいた方がよい。
        ```
        // git merge --no-ff topic 実行後
            o---o    :topic,
          /       \
        -o --------o :master <- HEAD
        ```

## マージの取り消し
* (参考) http://www-creators.com/archives/2111
* HEADの参照一覧を表示
    ```
    git reflog
    18828ad HEAD@{0}: merge topic: Merge made by the 'recursive' strategy.
    1c17f4f HEAD@{1}: checkout: moving from topic to master
    4f07825 HEAD@{2}: commit: Add header.
    1c17f4f HEAD@{3}: checkout: moving from master to topic
    1c17f4f HEAD@{4}: commit: New comit .
    3b1e279 HEAD@{5}: commit (initial): Inital
    ```
    * 上記の例では`HEAD@{1}`がマージの直前の状態を表しているので、`git reset`を使って強制的に参照・インデックス・作業ツリーすべての状態を戻す。
        ```
        git reset --hard HEAD@{1}
        ```
* revertを使って取り消す方法もあるが、あまり使わない方がよさそう(?)。データは元に戻るが、マージによる履歴は残ってしまうらしい。
    * (参考) http://www-creators.com/archives/2111

## コンフリクトしても強制的にマージ
* 作業ツリーやインデックスの内容は消えるので注意。
* 現在のブランチを強制的に指定したブランチに合わせる
    ```
    git reset origin/master --hard
    ```
* `git merge --theirs`といったコマンドもあるが、新しいコミットが作られてしまうためNG。
* オプションによる違い
    * `--soft`：現在のbranchの先頭だけをセット
    * `--mixed`（デフォルト）：現在のbranchの先頭とインデックスをリセット
    * `--hard`：現在のbranchの先頭、インデックス、作業ツリーを全部リセット

## コンフリクトの解決
* (参考) https://qiita.com/nfnoface/items/8823bfb8f50c4c90412d
* `git status`でconflictしたものを確認する。コンフリクトしていないものはstagingされているが、コンフリクトしているものは`both modified: 〜`で出てくる。
* 対象のファイルを修正する。
    ```
    <<<<<<< HEAD
    自分の環境の変更点
    =======
    マージを試みた他の環境での変更点
    >>>>>>> [commit id]
    ```
* 対象のファイルをaddし、commitを実施。


# git rebase
* (参考) https://qiita.com/takke/items/3400b55becfd72769214
* コミットする前に`git rebase -i`でまとめると、きれいになりそう。
    * すでに`git push`してしまった内容であれば`git push -f`で戻す。
    * ただし、他の人も使っている共有ブランチでは`git push -f`は絶対にやってはいけない。
* (IMO) そもそもコミットメッセージをきれいにする必要はあるのか、という気はする。プルリクエストのdiffで見れば十分ではないか。

## 過去のコミットメッセージを修正する
* (参考) https://qiita.com/kenose0328/items/185f7e8634d816c85a84

## authorやemailを編集したい場合
* 以下のやり方で可能。ただし、コミット日時も変わってしまうため、やるなら直近のコミットだけにする。既にpushしている場合はpush -fすることになるが、これも基本的にはやらない方がよい。
* 手順
    1. `git rebase -i <修正したいコミットの一つ前のコミットID>`
        * 一覧が出てくるので、修正したいコミットの「pick」を「edit」に変える。
    2. 修正したい分だけ以下を繰り返す。
        ```
        git commit --amend --reset-author
        ```
        * 編集画面が開くので問題なければそのまま保存。
        ```
        git rebase --continue
        ```
        * 次のコミットへ進む(?)。最後のeditであればそこで処理が終了しsuccessになる。
        * 過去に手動でマージ処理などをしている場合は、もう一度同じことをやり直す必要があるので注意。
    3. `git log`で修正されたか確認。
    4. 既にpushしてしまった場合は`git push -f`（あまりおすすめしない）。
* (参考) https://tech-1natsu.hatenablog.com/entry/2018/10/19/021855
* (参考) https://zenn.dev/kurun/articles/9f16a8a900f19e792321


# git tag
* `git tag <タグ>`
    * 例：`git tag -a v1.2 -m 'version 1.2' 9fceb02`
* 既存のタグ（が指すコミット）に対して他のタグをつけることもできる
    ```
    git tag <既存のタグ> <他のタグ>
    ```
* `git push origin <タグ>`
* (参考) https://qiita.com/growsic/items/ed67e03fda5ab7ef9d08

## タグの削除・変更
* 前提：基本的に公開されて使われているリポジトリでは、リモートにpushしたタグの変更をしてはいけない。
* 名前の変更：タグの名前は変更できないため、変更する場合は削除して新たにつけ直す必要がある。
* 紐づくコミット番号の変更
    ```
    git tag -f <タグ名> <コミット番号>
    git push -f <タグ名>   # リモートの方も変える場合
    ```
* タグの削除
    ```
    git tag -d <タグ>
    git push -d origin <タグ>   # リモートの削除
    ```
* (参考) https://gist.github.com/devlights/0a53bec59d1872a5327605f67a51ce66
* (参考) https://tec.tecotec.co.jp/entry/2022/12/14/000000
* (参考) https://qiita.com/growsic/items/60928fc67c9efe373a73


# cherry-pick
* 他のブランチのコミットの一部を取り込む。
    ```
    git checkout feature
    git log --oneline
    git checkout master
    git cherry-pick <コミットID>
    ```
    * (参考) https://rfs.jp/server/git/gite-lab/git-cherry-pick.html
* マージコミットを取り込む場合は`-m`をつける。
    * マージコミットは2つのブランチを統合しているため、どちらのブランチの流れを取り込むか番号で指定する必要がある。
        ```
        git cherry-pick -m 1 <commit_id>
        ```
    * (参考) https://www.ted027.com/post/git-cherry-pick/


# プルリクエスト
* プルリクエストとは、簡単に言うとマージしてほしいというリクエスト。
* pushした後、BitbucketやGitHubなどのサービスの機能（CLIツールを入れればコマンドラインからも可能(?)）でプルリクエストを出す。
* 他のメンバーにレビューしてもらい、以下の流れになる。
    * 修正が必要なら修正して再度push。
    * 不要ならマージしてもらう。
* WIP PR：Work in progress。開発中のものをPRすることで早期にレビューを実施でき、修正コストが比較的低くなるメリットがある。
    * (参考) https://www.kimoton.com/entry/2018/08/07/084134

## featureブランチ→masterのPRでマージしたものを取り消したい場合
* revertで戻す。
* ただし、その状態のままだと再度featureからマージできなくなってしまうため、featureブランチ側でそのrevertを取り込む。
* revertを取り込むとfeature側の修正も消えてしまうため、今度はrevert自体をさらにrevertする。
* (参考) https://satomikko94.hatenablog.com/entry/2014/06/30/235603
* 特定のコミットを消す
    ```
    git revert <コミットのハッシュ値>
    git revert <commit> --no-edit   # メッセージ編集画面を開かない
    ```
    * (参考) https://qiita.com/chihiro/items/2fa827d0eac98109e7ee
