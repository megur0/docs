- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 注
* 本メモは IMO(In my opinion)となる。


#  拡張, 実装, class修飾子の使い分け

## class修飾子/mixinの使い分け
![](./svg/class_usecase/class_judge.drawio.svg)

* プロパティ
    * 共通で持たせたいプロパティが複数のパターンがある場合、以下の方法を使い分ける。
    * パターン分のクラスを複数用意して、用途に応じたクラスを継承させる方法
    * ひとつのクラスの中でnullableなプロパティを継承する側で取捨選択させる方法

## extends, with, 移譲の使い分け
| | コンストラクタの再利用| 多重継承・実装 | 列挙型からの利用 | 本システムでの主な用途 | 本システムでの主な用途(テスト) |
|---|---|---|---|---|---|
|extends| o | x | x|class利用時(interface以外) | テストのスタブ(モック)化 |
|implements| o | o | o |interface class利用時 | テストのスタブ(モック)化<br/>※プライベートコンストラクタとなっていて継承できないもの |
|with| x | o | o | mixin利用時 | - |
|移譲| - | - | - | 責務が外にあるもの・外に出したいもの| - |
|移譲(インジェクション)| - | - | - |責務が外にあるもの・外に出したいもの<br/>※テストでスタブ化する必要があるもの| - |

## 列挙型の活用
* 利用可能なら積極的に使いたい。
* メリット
    * 羅列（switchによる網羅的な分岐、valuesで網羅的な処理ができる）
    * インスタンス生成不要でconst値として使える
* 利用できないケース
    * extendsが必要な場合
        * したがって親がbase class(implementsができない)の場合は利用できない。
    * 定数以外の引数が必要な場合

## too much　な 継承 の利用方法
* 極端な例ではあるが、下記のような継承の利用は冗長となる。
```
class ScreenA {void checkAuth(){〜};}
class ScreenB extends ScreenA { someFunc(){ super.checkAuth(); } }
```
* デメリットとして
    * 本来再利用したい機能以外の依存関係ができてしまう
    * 認証機能の責務を分離する機会を逃している
* 例えば別クラスとして外出しする方が良いだろう
```
class ScreenA(Auth auth) {〜}
class ScreenB(Auth auth) {〜}
```
