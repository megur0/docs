---
updated: 2026-07-24
---

[TOP(About this memo))](../README.md) > [一覧(Git)](./README.md) > 基本用語・概念


# 用語
* Gitには、リモートリポジトリ・ローカルリポジトリ・作業フォルダ（ワークツリー）の3段階がある。
* リモートリポジトリ：インターネット上、あるいはその他ネットワーク上のどこかに存在するリポジトリ。
* ローカルリポジトリ：ユーザーが利用するために、自分の手元のマシン上に配置するリポジトリ。
* ブランチ：SVNと同様の意味。マスターブランチはmaster（SVNのtrunkに相当）。
* ワーク（作業）ツリー：Gitの管理下に置かれた、実際に作業をしているディレクトリ。
* インデックス（ステージング）：リポジトリにコミットする準備をするための場所。リポジトリとワークツリーの間にインデックスが存在する。
    * ファイルをコミットするには、まずここに登録する必要がある。
* HEAD：今いるブランチの最新のコミット。
* リモートトラッキング（追跡）ブランチ：リモートリポジトリにあるブランチを監視しているブランチ。`origin/****`のこと。`git fetch`してから`git merge origin/xxxxx`をするというのは、リモート追跡ブランチにリモートの変更を取り込み、現在のブランチに`origin/xxxxx`を取り込んでいるということ。
* 上流ブランチ：引数なしで`git pull`したときの対象になるブランチ。たとえば`master`をチェックアウトして`git pull`すると、自動的に`origin/master`の変更を取得する。ここで`origin/master`はローカルブランチ`master`の上流ブランチであり、`master`は`origin/master`を追跡（tracking）している、ともいう。
    * (参考) https://qiita.com/uasi/items/69368c17c79e99aaddbf
* `origin/master`と`origin master`の違い
    * `origin/master`とスラッシュが付く場合、ローカルのrefs領域を指す。
    * `origin master`とスペースが付く場合、サーバーを指す。

## 参考サイト
* (参考) https://qiita.com/kohga/items/dccf135b0af395f69144


# HEAD~、HEAD^、@の関係
* Gitのv1.8.5以降、「HEAD」のエイリアスとして「@」が用意されている。
* `git show <ハッシュ値>`のようにハッシュ値を直接指定してもコミットを参照できる。ハッシュ値は`git log`で確認できる。
* 親コミットが複数ある場合（マージコミットなど）は、`^`（キャレット）と`~`（チルダ）を以下のように使い分ける。単一の親をたどる限りではHEAD~1とHEAD^は同じ意味になる。
```
o(HEAD~3) - o(HEAD~2 or HEAD~1^1) -o(HEAD~1 or HEAD^) -HEAD
        /                           /
    ------o(HEAD~1^2)-----------
```


# ref
* 参照（refs）はGitにおけるSHA-1ハッシュ値を示す別名である。
* GitのオブジェクトやコミットはSHA-1ハッシュ値で管理されている。ただし毎回SHA-1ハッシュ値を直接指定するのは手間なので、SHA-1ハッシュ値を指す参照（refs）が用意されている。
* (参考) https://senooken.jp/post/2020/09/12/

## .git/refsについて(TODO)
* `.git/refs/heads/`：ローカルのブランチへの参照を格納。
* `.git/refs/remotes/`：リモートリポジトリへの参照（ブランチ、タグ）を格納。
* `.git/refs/tags/`：ローカルのタグへの参照を格納。
* `origin/master`のように書いている場合、ローカルの`.git/refs/remotes/origin/master`を指している(?)。
* `.git/`以降の`refs`からを参照として使用でき、前半部分はGitが展開するため省略可能となっている。
    * 以下は同じ意味。
    ```
    git log refs/heads/master
    git log heads/master
    git log master
    ```

## refspec
* refspecはリモートリポジトリとローカルリポジトリの対応関係の書式。
* 具体的には`.git/config`内で指定され、`fetch`/`pull`と`push`で使用する。(TODO)
