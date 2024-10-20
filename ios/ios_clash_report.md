[TOP(About this memo))](../README.md) > [一覧(iOSアプリ開発)](./README.md) > クラッシュレポート


# WIP: 随時更新
* このメモは執筆中のため随時更新 

# 公式
* https://developer.apple.com/documentation/xcode/building-your-app-to-include-debugging-information

# 参考
* https://qiita.com/ichikawa7ss/items/9d53301c67449831a94e

# シンボル
* クラス名、グローバル変数、およびメソッドと関数名

# DWARF と DWARF with dSYM の違い
* DWARF(Debugging With Arbitrary Record Formats)
    * https://ja.wikipedia.org/wiki/DWARF
        > DWARF（ドワーフ）とは、広く使われているデバッグ用データフォーマットの規格である。当初ELFと共に設計されたが、オブジェクトファイルのフォーマットとは独立している
* XcodeにおけるDWARF と DWARF with dSYM の違い
    * https://stackoverflow.com/questions/22539691/whats-the-difference-between-dwarf-and-dwarf-with-dsym-file
    > 違いは、dSYMファイル付きDWARFの場合、アーカイブapp.xcarchiveには、クラッシュレポートでのコードの逆シンボリケーションに必要なdSYMファイルも含まれていることです。  
    > 一般的に、.xcarchive には以下が含まれています。   
        > dSyms  
        > Products  
        > info.plist  
    > したがって、配布用にアプリをアーカイブする際にクラッシュ レポートの外部分析が必要な場合は、dSYM ファイルで DWARF を使用する必要があります。  

# dSYM(デバッグシンボル)
* Xcodeがコードをマシン語へ変換する際にアプリのシンボルのリストを作成する
> この関連付けは、デバッグシンボルを作成するので、Xcode でデバッガを使用したり、クラッシュレポートによって報告された行番号を参照することができます。
> アプリのリリースビルドは、配布アプリのサイズを小さくするためにコンパニオンデバッグシンボル（dSYM）ファイルにデバッグシンボルを配置します。
> アプリの各バイナリファイル（メインのアプリ実行ファイル、フレームワーク、アプリ拡張機能）は、それ自身のdsymファイルを持っています。 
> コンパイルされたバイナリとそのコンパニオン dSYM ファイルは、ビルドされたバイナリと dSYM ファイルの両方によって記録されるビルド UUID によって結び付けられます。
> 同じソースコードから 2 つのバイナリをビルドしても、Xcode のバージョンやビルド設定が異なると、2 つのバイナリのビルド UUID は一致しません。
> バイナリと dSYM ファイルは、同一のビルド UUID を持つ場合にのみ互換性があります。 
> 配布する特定のビルドのためにdsymファイルを保持し、クラッシュレポートから問題を診断するときに使用してください。

# Configurationsによる違い
* XCodeではデフォルトで以下の２つのConfigurationsが設定されているが、デフォルトでReleaseでのみdSYMが生成される。
* Debug
    * デフォルトでコンパイルされたバイナリ ファイル内にデバッグ シンボルが配置
* Release
    * 配布されるアプリのサイズを縮小するために、デバッグ シンボルがdSYMファイルとして配置される。
* 設定は、Runner > Build Setting > Debug Information Format 上で確認・変更できる。
> アプリを配布用にビルドする前に、デバッグ情報フォ​​ーマットのビルド設定が DWARF with dSYM File に設定されていることを確認してください。これにより必要なファイルが生成され、アプリのリリース後にクラッシュを診断できるようになります。