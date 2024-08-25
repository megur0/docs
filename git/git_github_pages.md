- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 公式
* https://docs.github.com/ja/pages

# Jekyll
* https://jekyllrb.com/
* GitHub PagesではJekyllという変換ツールが利用されており、これによってマークダウンがhtmlへと変換される。
    

# 注意点
* https://docs.github.com/ja/pages/quickstart
* 既にマークダウンで構成していれば、下記に従ってすぐにWEBサイト化できるが、以下の注意点がある。
* GitHub Pagesのデフォルトのマークダウンのパーサーがgithub.comで利用されているGFMと異なる。
    * GFM: GitHub Flavored Markdown
        * githum.comで使われているマークダウン
* したがって、githum.com上で問題なく表示されていたマークダウンにおいて、GitHub Pages上では崩れる、あるいは表示されないものがある。
* この回避策については後述する。
* (IME) 
    * 筆者はこのメモのリポジトリをGitHub PagesによってWEBサイト化した(24/7/16)
    * しかし、(もともとのマークダウンの書き方がキレイではなかったが)かなりの部分が影響を受けてしまった。
    * 特にソースコードを記述している箇所が、ソースコードのブロックとしてパースされなかった箇所が多かった。
    * これが(おそらく)危険なスクリプトとして、（公開後に一瞬にして）一部のディレクトリがGoogleから危険なページとして扱われるようになってしまった
    * 修正後も危険なページの表示は変わらず、Googleの解除フォームへ申請をしたがすぐには解消されなかったためリポジトリの名前を変えることで対応した。


# マークダウンのパーサーを変更する
* https://github.com/github/jekyll-commonmark-ghpages
* Github Pagesのデフォルトのマークダウンのパーサーはkramdownというものが使われている。
* パーサーをGFMに準拠させるには、リポジトリのルートに_config.ymlを作成して、下記のようにする。
    ```
    markdown: CommonMarkGhPages
    commonmark:
    options: ["UNSAFE", "SMART", "FOOTNOTES"]
    extensions: ["strikethrough", "autolink", "table", "tagfilter"]
    ```
* commonmarkはGitHub Flavored Markdownのベースとなる仕様
  * 参考
    * https://spec.commonmark.org/
* CommonMarkGhPages
  * https://github.com/github/jekyll-commonmark-ghpages
    > Jekyll用のGitHub Flavored Markdownコンバーター
* 各extentsionは以下のGitHub Flavored MarkdownのSpecに解説がある。
  * https://github.github.com/gfm/
* オプションはドキュメントを見つけることができなかったが、例えばUNSAFEの記述がなかった場合、本文中のhtmlコードは除外される。
  * 参考: https://budougumi0617.github.io/2020/03/10/hugo-render-raw-html/
* 参考
    * https://hidakatsuya.dev/2021/02/14/markdown-and-commonmark-and-ghm.html


# (参考)マークダウンのパーサーがkramdown(デフォルト)の状態で筆者のマークダウンで発生した問題
* これらは記録として、残しておく。
* なお、これらの問題はパーサーを先述した通りに変えたことで全て解消された。
* kramdown自体の問題という話ではなくGFMに準拠したパーサーとkramdownの違いによって発生した問題となる。
* kramdownのオプション等で回避できたかもしれないが、その点については調査していない。
## 親ディレクトリへの相対パスがうまく変換されない。
* Jekyllによってmdファイルはhtmlへ変換されるが、このときに各リンクのパスも合わせて変換される。
* 一方で、親ディレクトリ上のマークダウンへのリンクではファイルの拡張子が変わらない事象が確認された。
* 例えば`[リンク](../test.md)` は  `<a href="../test.md">` のようにmdのまま変換されてしまい、リンク切れとなってしまう。
* 原因としては、内部で以下のプラグインを利用しているようで、コレクション機能というものを有効にしないと親への相対パスが期待通りの動作をしない、ということがわかった。
* _config.ymlを作成してcollectionsをtrueとすることで、上記の問題が解決できた。(ただ、この方法が正しかったのか明確には分からない。)
    ```
    relative_links:
    enabled:     true
    collections: true
    ```
    * 参考
        * https://github.com/benbalter/jekyll-relative-links
## コードブロックとしてパースされないケース
* 1例として、下記のように箇条書きの後に同じインデントレベルのコードブロックがあるとうまく表示されない場合がある。
  * うまくパースされない(ことが多い)ケース
  ```
  ソースコード
  ```
  * うまくパースされるケース
    ```
    ソースコード
    ```
* ただ、上記の上手くパースされないケースは、内部のコードの内容によってはうまくパースされる場合があり、はっきりとした法則はわからない。
* インデントさえしていれば、問題ない。
## h2やh3が直前の箇条書きの中に入ってしまう
* 下記のように記述すると、サブタイトルとして生成されるh2タグが、箇条書きのliタグの中に含まれるようになってしまう。
  ```
  # タイトル
  * 箇条書き
  ## サブタイトル
  ```
* 以下のように改行する必要がある。
  ```
  # タイトル
  * 箇条書き

  ## サブタイトル
  ```
## 引用と箇条書き
* 以下のように、連続して引用の行と箇条書きの行を記述すると2が1にネストされてしまう。
  ```
  > 1
  * 2
  ```
* 以下のように間に改行をいれる
  ```
  > 1

  * 2
  ```


# テーマを利用する
* https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll
* 下記のように、`jekyll-theme-NAME` という命名規則で_config.yml へ記述することでサポートされているテーマを利用できる。
    ```
    theme: jekyll-theme-architect
    ```
    * サポートされているテーマ
        * https://pages.github.com/themes/
    * デフォルトのテーマはminimal
        * https://github.com/pages-themes/minimal
* その他のGitHub でホストされている外部のテーマを利用する場合は下記のように記述する。(THEME-NAMEが個々のテーマのREADMEにしたがう)
    ```
    remote_theme: THEME-NAME
    ```
* (IMO) 単純にコードの読みやすさ、という観点ではサポートされているテーマは、筆者個人としてはあまりユースケースに合わなかった。
    * github.comのメモの延長として利用する場合はminimalのままで、必要であればスタイルをカスタマイズする事が時間をかけずに済む方法かもしれない。

## (未調査)テーマのスタイルを自身でカスタマイズする。
* https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll#customizing-your-themes-css
* https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll#customizing-your-themes-html-layout
* minimalのカスタマイズ方法
  * https://github.com/pages-themes/minimal

# その他 _config.ymlの設定
* sitemap
  ```
  plugins:
    - jekyll-sitemap
  ```
* lang
  ```
  lang: ja
  ```

# Google Analytics
* https://github.com/pages-themes/primer#customizing-google-analytics-code
* _config.ymlへ以下を追記
```
google_analytics: G-XXXXXXXXXX
```
* _includes/head-custom-google-analytics.html を作成
```
{% if site.google_analytics %}
  <!-- Google tag (gtag.js) -->
  <script async src="https://www.googletagmanager.com/gtag/js?id={{ site.google_analytics }}"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', '{{ site.google_analytics }}');
  </script>
{% endif %}
```
* 参考
  * https://zenn.dev/key_luvless/articles/d6b14182c0b4e0


# (IME)Google サーチコンソールへの登録
* 筆者の場合は以下の手順で登録した。
* プロパティを追加 > URLプレフィックス追加 > Google Analyticsで認証
* サイトマップの登録
* 危険なコンテンツ判断されてしまった場合
  * ソースコードを大量に含んでいることが原因と考えられるが、危険なコンテンツを含んでいると、サーチコンソールで判定されてしまった（具体的なページなどは表示されない）
  * 修正が完了したことを報告できるので、そのフォームに「当該コンテンツはGithub Pagesによって作成した、ソフトウェア開発に関するコンテンツを扱ったWEBサイトでサンプルコードを多く含むものであり、ブラウザ上で動作するコードは一切含まない」という旨を記載して送信
  * 翌日にはGoogleより通知が届き、有害なサイトやダウンロードへのリンクが含まれていないことを確認できたので警告を削除する、という旨の通知メールが届く。