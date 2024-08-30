[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > 導入

# 公式ドキュメント
* https://docs.flame-engine.org/latest/flame/flame.html

# サンプルコード
* 目的別のサンプルコード
    * https://github.com/flame-engine/flame/tree/main/examples
    * ドキュメントの各コンテンツについて、動作するサンプルコードも多く含まれている。
* シンプルな完全な構成例
    * https://github.com/flame-engine/flame/blob/main/packages/flame/example/lib/main.dart
## サンプルコードのデモ
* https://examples.flame-engine.org/
* 右のメニューからソースコードへジャンプすることもできる
## サンプルコードの中の一つを手元で動かす
* Flutterプロジェクトを作成する
* flutter pub add flame
    * サンプルによって他にも必要なパッケージがある場合はそちらも加える
* assets, commonsフォルダの配置
    * flameリポジトリをダウンロード
    * examples/assetsをディレクトリごと手元のプロジェクトへコピーする
    * examples/lib/commonsをディレクトリごと手元のプロジェクトへコピーする
* assetsをpubspec.ymlへコピー
    ```
    assets:
        - assets/images/animations/
        - assets/images/
        - assets/images/tile_maps/
        - assets/images/layers/
        - assets/images/parallax/
        - assets/images/parallax/
        - assets/images/rogue_shooter/
        - assets/spine/
        - assets/svgs/
        - assets/tiles/
        - assets/audio/music/
        - assets/audio/sfx/
        - assets/yarn/
    ```
* サンプルコードをコピー
    * サンプルコードはFlameGameの派生クラスで作成されているため、GameWidgetに渡すことで実行できる
    * ただし、コードによってはGameWidgetではないウィジェットや、引数を渡す必要があるコードもあるため注意。
    ```dart
    void main() {
    runApp(
        GameWidget(
            game: サンプルコード内のFlameGameの派生クラス(),
        ),
    );
    }
    /*
    対象のサンプルコードをコピー
    */
    ```
* 実行

# 作成したゲームをGithub Pagesで公開する
* https://docs.flame-engine.org/latest/flame/platforms.html#deploy-your-game-to-github-pages

# ファイル構成
* https://docs.flame-engine.org/latest/flame/structure.html

