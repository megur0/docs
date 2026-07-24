---
title: "環境構築・リポジトリ管理"
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(Git)](./README.md) > 環境構築・リポジトリ管理


# help, version
* `git help commit`
* `git --version`


# config
## 設定の段階
* システム：インストールしたgitへの設定（オプション: `--system`）
* グローバル：ユーザー単位での設定（`--global`）
* ローカル：リポジトリ単位での設定（`--local`。ローカルのみオプション省略可能）

## 設定コマンド
```
git config --system [システムへの設定]
git config --global [グローバルへの設定]
git config --local [ローカルへの設定]
```
* 設定ファイルは直接編集することも可能だが、コマンドでの設定を推奨。

## 設定ファイルの場所
* システム：`/etc/gitconfig`
* グローバル：`~/.gitconfig`
* ローカル：作業フォルダの`.git/config`に書き込まれる。`vim .git/config`などで直接編集も可能。
* gitコマンドではシステム、グローバル、ローカルの順に設定が読み込まれる。
* Macの場合、何も設定していない状態ではシステム・グローバルの設定ファイルは存在しない（何かしら設定コマンドを実行すると作成される）。

## 初期設定
* ユーザー名・メールアドレスは必ず設定しておく。公開リポジトリだとWebから見えてしまうので注意。
    * `vi .git/config`で直接編集するか、以下のコマンドで設定する。
        ```
        git config --local user.name "xxxxxx"
        git config --local user.email "xxx@xxx.com"
        ```
* メールアドレスは、GitHubの[Emails設定](https://github.com/settings/emails)にある「Keep my email addresses private」で案内されるプライベート用アドレスを使うのがよいだろう。
    * (参考) https://qiita.com/ucan-lab/items/aadbedcacbc2ac86a2b3 （GitHubでメールアドレスを表示しない方法）
* リモートURLを変更したい場合
    ```
    git config --local remote.origin.url "https://xxxxxx@github.com/xxxxxxx/needs.git"
    ```

## 設定ファイルの確認
```
git config --local -l   # ローカルの設定
git config --global -l  # グローバルの設定
```

## unset
```
git config --local --unset user.name
git config --local --unset user.email
```

## (参考) その他のおすすめ設定
* (参考) https://qiita.com/hayamofu/items/d8103e789196bcd8b489
* `core.quotepath`：diffなどのファイルパスを出力するコマンドで（`-z`オプションを付けない場合）、パス中の特殊文字をバックスラッシュ付きのダブルクォートで囲む機能。OFFにしても問題なさそう。
```
# パーミッションの変更を無視する
git config --local core.filemode false
# ファイル名の大文字・小文字の変更を検知する
git config --local core.ignorecase false
# カラー設定
git config --local color.ui true
git config --local color.diff auto
git config --local color.status auto
git config --local color.branch auto
# 日本語ファイル名をエスケープせずに表示
git config --local core.quotepath false
# 濁点つきのファイル・ディレクトリが分けて表示されてしまうUTF8-MAC問題の対策
git config --local core.precomposeunicode true
```


# clone
* cloneは、リモートリポジトリが保持しているデータをほぼすべてコピーする。プロジェクトの全ファイルの全履歴が、`git clone`で手元にやってくる。
* 指定のディレクトリに入れたい場合
    ```
    git clone <リモート> <ローカルのフォルダ>
    ```
    * すでに存在するディレクトリで、かつ中身がある場合はエラーになる。
* ブランチを指定してclone
    ```
    git clone -b <ブランチ名> <リポジトリのアドレス>
    ```


# remote
## pushとリモート名
* pushはアップロード。`git push <name> <branch>`で、ローカルのコードを`<name>`という名称のリモートリポジトリの`<branch>`ブランチにアップロードする。`<name>`は「サーバーを表すただの短縮名」なので、URLでも良い。
    * 例：`git push origin master`
    * 短縮せずに全部書くと`git push git@github.com:DQNEO/sample.git master:master`のようになる。`master:master`は、ローカルのmasterをリモートのmasterに反映させるという意味。
* リポジトリの別名（name）の一覧は`git config --list`で確認できる。
    * 例
        ```
        remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
        remote.origin.url=https://github.com/ユーザー名/レポジトリ名.git
        ```
    * (参考) https://qiita.com/hshimo/items/99811144bf4a081319e5

## 基本
* リモートリポジトリの一覧を表示（`-v`で詳細を表示）
    ```
    git remote
    ```
* リモートリポジトリの追加
    ```
    git remote add <name> <url>
    ```
    * `<name>`はリポジトリの場所（URL）の別名。
    * 例（GitHubの場合）
        ```
        git remote add origin git@github.com:BabaShun/sample.git
        ```
        * 慣例上「origin」とすることが多い。削除は`git remote rm origin`。
    * このコマンドは「`<name>`という短縮名でリモートリポジトリを登録する」という命令なので、短縮名を使わずURLを直接コマンドに渡す（例：`git push <URL> <ブランチ>`）ことも可能ではある。便利なので通常は短縮名を使う。

## リモート側で名前が変更された場合
* (参考) http://mzgkworks.hateblo.jp/entry/git-remote-changename
```
git remote set-url origin <新しいURL>
git remote -v   # 確認
```

## リモートリポジトリの作成
* その1：GitHubやBitbucketでリモートリポジトリを作成し、cloneして必要なものを入れてpushする。
* その2：既存のディレクトリをpushしたい場合
    ```
    git init
    git add .
    git commit -m "Initial commit"
    git config --local user.name "xxx"
    git config --local user.email "xxx@xxx.com"
    git remote add origin https://github.com/XXXX/XXXXXX.git
    git push -u origin master
    ```
* 既にあるリポジトリをベースに新しいものを作りたい場合
    * 新しいリポジトリを作成してcloneする。
    * ベースにしたいものを`git archive`でエクスポートする（[エクスポート](#エクスポート)参照）。
    * エクスポートしたものをcloneしたディレクトリに入れてadd/commit/push。

## (参考) ローカルでpush可能なリポジトリを作成する場合
* (参考) http://blog.atwata.com/tool/2017/10/10/init-local-bare-git-repository.html
* 作成
    ```
    cd /path/to
    mkdir myapp.git   # リモートリポジトリのディレクトリは.gitをつけるのが慣例
    cd myapp.git
    git --bare init --shared
    ```
    * `--bare`：作業ファイルを持たない、pushされるための専用リポジトリ（最小限のリポジトリ。ベアリポジトリという）。
    * `--shared`：リポジトリを共有可能にするためのオプション。これがないとpushしてもファイルを作成できない。
    * `ls`でファイルが作成されていることを確認できる（例：`branches config description HEAD hooks info objects refs`）。
* clone（ローカル）
    * cloneはSSHを使うため、ローカル内で完結する場合でもSSHが必要(?)。
        ```
        cd /tmp/
        git clone localhost:/path/to/myapp.git
        ```


# mirror(WIP)
* 用途は要確認(?)。
* `git push --mirror`
    * ローカルリポジトリの`refs/`がそのままリモートリポジトリの`refs/`にプッシュされる。
* `git clone --mirror <URL>`
    * 以下と等価らしい(?)。
        ```
        git init --bare
        git remote add --mirror=fetch origin <URL>
        git fetch origin
        ```


# SSH
## sudoでのgit cloneの注意点
* sudoで`git clone`（SSH経由）を実行するとrootで実行されることになるため、秘密鍵が想定しているユーザーのホームディレクトリ以外（例：`/home/ec2-user/`ではない場所）から参照されてしまうことがある(?)。

## GitHubのSSH設定（注意点のみ）
* `~/.ssh/config`に以下のように設定する。
    ```
    Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/sample/id_rsa
    ```
* Hostを単に「github」とだけ設定した場合、pushする際に`git@github.com:sampleProject/aaaa.git`の`github.com`部分を`github`に変える必要がある。

## SSH時のパスフレーズとGIT_SSH_COMMAND
* パスフレーズは作成時に空欄にしておく方が扱いやすい。
* 秘密鍵の認証に失敗した場合、パスフレーズ入力の対話処理に入ってしまうことがあり、CIで実行すると無応答になってしまう。対話処理を出さず、失敗時は異常を示す終了コードを返したい場合は、事前に`GIT_SSH_COMMAND="ssh -o BatchMode=yes"`を設定してバッチモードにすればよい。
    * 例
        ```
        GIT_SSH_COMMAND='ssh -F /home/ec2-user/.ssh/config -i /home/ec2-user/.ssh/id_rsa -o BatchMode=yes' git clone ...
        ```
    * (参考) https://zenn.dev/amay077/articles/ef1016062f5676


# fork
* Gitの機能ではなく、GitHubなどのサービス側の機能。
* 複製した側のリポジトリから元リポジトリにプルリクエストを送ることができる。
    * (参考) https://takemikami.com/archives/1451/
* cloneと違い、forkすると元リポジトリの管理者に通知が行く。
    * forkという行為はオリジナルへの貢献を前提とする。オリジナルに貢献する意図がない場合はforkする必要はなく、読み込み専用のURLで`git clone`するだけでよい。
    * (参考) https://yasulab.tumblr.com/post/11856357434/fork-%E3%81%99%E3%82%8B%E6%84%8F%E5%91%B3-github-%E3%81%AB%E3%81%AF-fork-%E3%81%A8%E3%81%84%E3%81%86%E6%A9%9F%E8%83%BD%E3%81%8C%E3%81%82%E3%82%8Bfork-%E3%81%AF-git
    * (参考) https://qiita.com/iverson3kobe0824/items/3e95cc5476b1c621fe95

## forkを使ったフロー
* cloneは単に任意のリポジトリをローカルに複製するもの。
* forkはOSSなど自分以外のリポジトリに対して、追加機能の実装やバグ修正を行いたい場合に使用する。
* 例
    1. リポジトリAをfork
    2. forkしたリポジトリBをローカルにclone
    3. cloneしたリポジトリCで開発し、リポジトリBに反映
    4. リポジトリAの管理者にPull Requestを送信
* (参考) https://qiita.com/matsubox/items/09904e4c51e6bc267990

## fork元の変更を取り込みたい場合
```
git remote add <適当なname> <fork元のURL>/xxx.git
git fetch <適当なname>
git checkout master
git merge <適当なname>/master
```
* (参考) https://qiita.com/Nossa/items/ace2ab802adc85f86b20


# 履歴まで含めてリポジトリをコピーしたい場合
* 方法1：fork
    * forkすると元リポジトリに通知が行くため、単純にコピーしたいだけの場合には向かない。本来forkは元リポジトリへの貢献を前提とした使い方であるため、用途としてもずれる。
* 方法2：GitHub importを使う
    * 動作が遅いらしい(?)。
    * https://docs.github.com/ja/github/importing-your-projects-to-github/importing-source-code-to-github/importing-a-repository-with-github-importer
* 方法3：`git remote add <好きな名前> <コピー元のURL>`としてから`git push -f <好きな名前> <ブランチ名>`する
    * ただし、これだと各ブランチを個別に反映する必要がある。
    * (参考) https://qiita.com/developer-kikikaikai/items/6a43978f3eee3c657310
* 方法4（最も簡便）：`--bare`をつけてcloneすると、gitファイルだけを取得できるので、そこから`git push --mirror <コピー先リポジトリ>`で全てをpushする
    * このやり方だと全ブランチ・全履歴をまとめてコピーできる。
    * (参考) https://qiita.com/ossan_pg/items/6e63ba0ae6e08a056a2f
    * (TODO) 方法4でオリジナル側の変更を後から取り込めるかは未確認。forkの節にある`git remote add`のやり方と同様にできる可能性がある。


# エクスポート
* (参考) http://dqn.sakusakutto.jp/2012/11/git_export_archive_checkout_index.html
* 除外ファイルを作っておく
    * `.gitattributes`ファイルを作成し、以下を記述する（例）。
        ```
        .gitignore export-ignore
        .gitattributes export-ignore
        ```
        * `.gitignore`に書いてあるファイルは記載不要（`.DS_Store`やideaディレクトリなど、`.gitignore`側に記載済みであれば不要）。
    * 除外設定を反映するため、`git add`、`git commit`しておく。
* エクスポート実行
    ```
    mkdir /path/to/tmp
    git archive master | tar -x -C /path/to/tmp
    ```
    * `-x`：解凍、`-C`：解凍先


# submodule(WIP)
* (参考) https://qiita.com/aki_55p/items/ed3f1d77b3a7235c8bec
* Gitプロジェクトの中に別のGitプロジェクトを入れる仕組み。
    * 例えば`hoge`というリポジトリの中で、`fuga`というリポジトリを外部から`git clone`してきた場合、`hoge`側で`git push`しても`fuga`の中身はpushされない。
    * このようなケースではsubmoduleを使う。
        ```
        git submodule add <外部リポジトリ> <ローカルで格納したいディレクトリ>
        ```


# subtree(TODO)
