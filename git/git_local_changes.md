---
title: "ローカルの変更管理 - Git"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(Git)](./README.md) > ローカルの変更管理


## status
* `git status`
* 以下がわかる。
    * 今どのブランチにいるか
    * 今のブランチの上流ブランチは何か
    * ローカルの変更 など
* 例
    ```
    On branch test
    Your branch is up to date with 'origin/test'.
    ....
    ```
* マージする前などは、作業ツリーの状態を確認して、必要に応じて`git stash`で退避する。
* 除外ファイルの確認
    ```
    git status --ignored
    ```


## git diff(WIP)
* (参考) https://qiita.com/shibukk/items/8c9362a5bd399b9c56be
* `git diff --stat`
* `git diff HEAD..<リモート名>/<ブランチ名>`
    * 手元のブランチの最新（HEAD）とリモートブランチを比較。
* `git diff <コミットID1>..<コミットID2>`
* `--name-only`でファイル名のみ表示。
* `git diff <コミットID1> <コミットID2> <ファイル名>`でファイルを指定。
* `git diff <ブランチ名> <ブランチ名>`でも比較可能。
    * チェックアウトしていないブランチも指定できる（例：`git diff develop-ga origin/staging`）。
    * ただし`staging`をチェックアウトしていない場合、`git diff develop-ga staging`のようにローカルブランチ名で指定するとエラーになるはず。
* 作業ツリーとの差分を確認
    ```
    git diff HEAD          # git diff だけでも同じ
    git diff HEAD --stat   # 詳細ではなく、どの程度変更があったかを見る
    git diff HEAD <branch> # 他のブランチと比較（例：git diff HEAD master --stat）
    ```
* リモートブランチとの変更点を見る
    ```
    git diff origin/test --stat
    ```


## add, commit
### 覚えておきたいこと
* 直前のコミットに上書きする
    ```
    git commit --amend                # コミットメッセージ入力のためにエディタが開く
    git commit --amend -m "fugafuga"  # 直前のコミットのメッセージを修正
    ```
    * `git add`済みの内容があれば、それも一緒にコミットされる(?)。
    * (参考) http://ism1000ch.hatenablog.com/entry/2014/03/26/190939
* addを取り消す場合
    ```
    git reset HEAD
    ```
* `-v`（`--verbose`）：メッセージを入力する際に下部に差分を表示してくれる（この部分および先頭に`#`がある行はコミットされない）。
* `-a`を付けるとaddを省略してcommitできる。

### 基本
* commit：ローカルリポジトリに変更を反映するのがcommit。リモートに反映するのがpush。
* コミットの流れ
    ```
    git add .
    git status                # 確認
    git commit -v
    git commit -m "ああああ"      # ヘッドメッセージだけ書く
    git commit -a -m "あああああ"  # addを省略
    ```
* `git add`
    * `git add`によってGitのコミット対象（ステージ）にする（インデックスに登録する）。
        * 例：`git add Program01.java`、`git add *.java`、`git add dir/*.java`
        * `git status`で見ると、「Changes to be committed（変更対象）」「Untracked files（コミット対象外）」が確認できる。
        * (参考) https://eng-entrance.com/git-add
    * ただし、削除したファイルは`add`でコミットされないので、ファイルを消した場合は`git rm <ファイル名>`のようにするか、`git add -u`とする（`-u`は変更があったファイルのみ更新するオプション。新規追加したファイルはaddされない）。
    * `git commit -a <ファイル名>`とするとaddも一緒にやってくれる。
* 一時的にnameやemailを指定する場合
    * `git commit --author='ユーザー名' -m "..."`という方法もあるが、コミッターとauthorは別のものらしく、上記だと`.git`内に設定されているコミッター情報もGitHub上に一緒に出てきてしまう。以下の方法の方がよい(?)。
        ```
        git -c user.name='xxxxxx' -c user.email='<メールアドレス>' commit -m '...'
        ```
* コミットメッセージ
    * メッセージはコミット時に必須。標準的には以下の形式で書く。
        * 1行目：コミットでの変更内容の要約
            * fix：バグ修正
            * hotfix：クリティカルなバグ修正
            * add：新規（ファイル）機能追加
            * update：機能修正（バグではない）
            * change：仕様変更
            * clean：整理（リファクタリング等）
            * disable：無効化（コメントアウト等）
            * remove：削除（ファイル）
            * upgrade：バージョンアップ
            * revert：変更取り消し
        * 2行目：空行
        * 3行目以降：変更した理由
    * 例
        ```
        [fix]削除フラグが更新されない不具合の修正

        refs #110 更新SQLの対象カラムに削除フラグが含まれていなかったため追加しました。
        （refs #110はRedmineのチケット番号）
        ```
    * (参考) https://qiita.com/itosho/items/9565c6ad2ffc24c09364
    * (参考) その他のコミットメッセージのカテゴリー例
        ```
        # 初めてのコミット（Initial Commit）
        # バージョンタグ（Version Tag）
        # 新機能（New Feature）
        # バグ修正（Bugfix）
        # リファクタリング(Refactoring)
        # ドキュメント（Documentation）
        # デザインUI/UX(Accessibility)
        # パフォーマンス（Performance）
        # ツール（Tooling）
        # テスト（Tests）
        # 非推奨追加（Deprecation）
        # 削除（Removal）
        # WIP(Work In Progress)
        ```
    * (参考) タグ・ラベルのカテゴリー例
        ```
        enhance     # 機能強化（improve）
        refactoring # 外部的振る舞いを保つ
        remake      # 機能やアートワークを作り変える
        experiment  # 実験的な試み
        bug, bugfix # fix
        documentation
        ```
        * (参考) https://docs.github.com/ja/issues/using-labels-and-milestones-to-track-work/managing-labels
        * (参考) http://negi-lab.blog.jp/MyLabelsForIssue

### その他
* 初回のコミットを取り消す場合
    ```
    git update-ref -d HEAD
    ```
    * (参考) http://tweeeety.hateblo.jp/entry/2017/04/18/000621
* addを取り消す場合
    ```
    git rm (-r) -f --cached .
    ```
    * `git reset HEAD`（`git reset --mixed HEAD`）でも良いが、初回コミット前はHEADが何も参照していないためエラーになる。
    * 一部だけ取り消す場合
        ```
        git rm --cached <ファイルパス>
        git reset HEAD <ファイルパス>
        ```
    * ファイルごと消す場合
        ```
        git -f rm <ファイルパス>
        ```
        * まだインデックスにaddしていない場合は`-f`は不要。


## rm
* ファイルを削除したい場合
    * IDEによっては、ファイルを削除すると自動的に以下のコマンドが実行されることがある(?)。
    ```
    git rm --cached <ファイルパス>  # インデックスからのみ削除（ファイル自体は残す）
    git rm -f <ファイルパス>        # ファイルごと削除（例：git rm -f path/to/.DS_Store）
    git rm -r --cached <フォルダ>   # フォルダ丸ごとインデックスから削除
    ```


## gitignore
* 例
    ```
    .idea
    .DS_Store
    db.ini
    ```

### .gitignoreが反映されないとき
* 既にGitに追加済みのファイルの場合、以下の手順で対応する。
    1. `.gitignore`を編集
    2. キャッシュを削除：`git rm (-r) --cached <ignoreしたいファイル>`
    3. commit & push
* (参考) https://qiita.com/fuwamaki/items/3ed021163e50beab7154


## reset
### 覚えておきたいこと
* コミットを一つ前に戻す
    ```
    git reset --soft HEAD^
    ```
* 特定の状態にファイルも含めて戻す
    ```
    git reset --hard <ハッシュ値>
    ```

### 基本
* 特定の状態に戻したい場合（コミット単位）
    ```
    git reset --hard HEAD   # 全部戻るので注意
    ```
    * `--soft`：HEADのみ
    * `--mixed`：HEAD/インデックス
    * `--hard`：HEAD/インデックス/ワークツリー
* ローカルのコミットをリモートの状態に戻したい場合
    ```
    git reset origin/xxx
    ```

### untrackedなファイルの削除
* `git reset --hard`をしてもuntrackedなファイルは残る。削除するには以下を使う。
    ```
    git clean -fdx
    ```


## checkout
### 覚えておきたいこと
* 特定のファイルを最後のコミットに戻す
    ```
    git checkout HEAD <filepath>
    git checkout <hash> <filepath>
    ```

### 基本
* (参考) http://www-creators.com/archives/1388
* SVNのcheckoutとは違う機能なので注意（SVNのcheckoutはgit cloneに近い）。
* `git checkout`の機能は以下の2つ（別の機能なので注意）。
    1. 作業ブランチを切り替える
    2. 指定したコミットもしくはインデックスの状態を作業ツリーに展開する
    * ブランチ名を指定すると1の機能、ファイルパスやファイル名を指定すると2の機能になる。
* `git checkout <コミット指定> -- <ファイル名>`
    * `--`は省略可能。`git checkout HEAD -- master`のように「master」というファイルを作業ツリーに展開したい場合などのために、この文法になっていると思われる(?)。そのような状況はまずないだろう。
    * 例1：コミット`6f87gs1`のindex.phpを作業ツリーに展開
        ```
        git checkout 6f87gs1 index.php
        ```
    * 例2：HEADのindex.phpを作業ツリーに展開
        ```
        git checkout HEAD index.php
        ```
        * コミット指定を省略すると、インデックスから作業ツリーへ展開される。
* 新規ブランチをチェックアウト
    ```
    # 現在のブランチの先頭から、new-feature ブランチを作成してチェックアウト
    git checkout -b new-feature

    # masterを起点に、favorite-featureブランチを作成してチェックアウト
    git checkout -b favorite-feature master
    ```
    * これは`git branch <ブランチ名>`と`git checkout <ブランチ名>`を短縮して行っているコマンド。
    ```
    # devブランチがすでに存在しても、強制的にmasterに合わせてチェックアウトする
    git checkout -B dev master
    ```
    * 内部動作は要確認(?)。既存のdevの中身がどうなるのかは未確認。
* 強制的に切り替える
    * 通常はインデックスや作業ツリーにまだコミットしていない変更内容や新規追加ファイルが残っていればエラーになる。変更を全て破棄してよければ以下を実行する。
        ```
        # インデックスと作業ツリーの変更を破棄して、強制的にチェックアウトする
        git checkout -f master
        ```
    * インデックスや作業ツリーを退避させる`stash`というコマンドもある。


## stash
* 退避
    ```
    git stash   # save は省略可能
    ```
    * `git status`で、変更したファイルがなくなっていることがわかる。
    * ただし、デフォルトではワーキングツリー内の「新規ファイルの追加」は退避対象に含まれない。含める場合は以下。
        ```
        git stash -u
        ```
        * 以前は`git stash save -u`という書き方だったが、`-u`の位置が変わったらしい(?)。
* 一覧・内容確認
    * `git stash list`でstashした一覧が見られる。`<stash名>: WIP on <stashを行ったブランチ名>: <ハッシュ> <コミットコメント>`という形式で出る（ハッシュ・コミットコメントはstashを行った時点のHEADのもの）。
    * `git stash list -p`で変更内容の詳細が見られる（`git log`のオプションが使える）。
    * `git stash show <stash名>`で各変更が見られる。
* 復活
    ```
    git stash apply stash@{0}
    git stash pop stash@{0}   # 復活＆listから削除
    ```
    * スタッシュ名を省略すると直近のstashが対象になる。
* listから削除する
    ```
    git stash drop <消したいstash名>
    ```
* 復活を取り消し
    ```
    git stash show <適用したstash名> -p | git apply -R   # -pでパッチ形式出力、-Rは逆適用
    ```
* (参考) https://qiita.com/fukajun/items/41288806e4733cb9c342
* (参考) https://www.granfairs.com/blog/staff/git-stash


## git log
### 覚えておきたいこと
```
git log -1 --stat
git log --oneline -2
git log -p css/style.css
git log --oneline           # ワンラインで表示。ハッシュ値も短縮表示される(?)。git diffやgit showと組み合わせて使うと良さそう(?)。
git log --graph             # グラフ表示
git log --oneline --graph   # ワンライン＋アスキーグラフ表示
git log -p -2                # -p: 詳細のdiff
git log --name-status        # 変更したファイルを表示
```

### その他
```
git log --since="4 days ago" --until="2015/01/22"
git log --author='xxxxxx'
git log origin/master --oneline -2
```


## git show
* `git show`で開始、`q`で終了。現在のブランチで一つ前のコミットとの比較を表示する。
* ハッシュ値やファイル指定ができる。
    ```
    git show <ハッシュ値>
    git show <ハッシュ値>:<file-path>
    ```
* コミットログ確認
    ```
    git show HEAD     # 最後のコミット
    git show HEAD^    # 最後の1つ前
    git show HEAD^^   # 最後の2つ前
    git show HEAD~2   # 最後の2つ前（同じ意味）
    ```


## git clean
* (参考) https://gotohayato.com/content/104/
* 作業ディレクトリから追跡対象外のファイルを削除するコマンド。
* `git clean -ndx`：ドライラン
* `git clean -fdx`：すべて削除（新規作成ファイルも消えるので注意）
* `git clean -fX`：ignoreファイルのみ削除
* その他のオプションは`git clean -h`で確認。
