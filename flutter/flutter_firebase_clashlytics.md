[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > Firebase Clashlytics


# 導入(iOS)
* 以下の手順に従う
    * https://firebase.google.com/docs/crashlytics/get-started?platform=flutter&hl=ja#add-sdk
* 上記の手順を実行するとClashlyticsは、ビルドフェーズに以下の名前の実行スクリプトを追加する。
    * `[firebase_crashlytics] Crashlytics Upload Symbols`
* 上記のスクリプトはdSYMファイルをClashlyticsへアップロードする。
    * https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=flutter&hl=ja
## デバッグビルド時にdSYMを送信する
* デフォルトでは、デバッグビルドでdSYMを生成しない点に注意。
    * この場合、Clashlytics上でdSYMの欠損エラーが表示される。
        * ※ ただし、クラッシュが発生した際に何度かFlutterアプリをリスタートをしているとdSYMの欠損エラーが解消してClashlyticsでレポートを見ることができた。
            * (IMO)もしかするとデバッグモードでも「クラッシュが発生した時はdSYMを送る」といった処理が内部で行われているかもしれない
    * XCodeで Runner > Build Setting > Debug Information Format で確認すると、デバッグビルドにおいては「dwarf」となっている。
* デバッグビルドも出力したい場合は、こちらを`dwarf-with-dsym`へ変更する。
* 参考: https://github.com/flutter/flutter/issues/116493#issuecomment-1340259052
    * ※ このissue自体は別の内容。(Flutter Framwork自体のdSYMが不足してClashlyticsの詳細が見れない、というissue)


# Dartコードを難読化した際のシンボルファイルはアップロード不要
* ClashlyticsはdSYMを上記のスクリプトによってアップロードするが、flutterのビルド時に難読化(`--obfuscate`)を行った場合でも、dSYMは難読化されない。
    * ※ もともとdSYMはiOSのリリースビルドではバイナリには含まれないため、dSYMが難読化される必要はない。
    * したがって`--split-debug-info`によって出力される.symbolファイルをClashlyticsへアップロードする必要はない。
* 参考
    * https://github.com/firebase/flutterfire/issues/8934#issuecomment-1521852271
    * https://github.com/firebase/flutterfire/issues/8934#issuecomment-1535457929
    * https://github.com/flutter/flutter/issues/124715
    * https://github.com/firebase/flutterfire/issues/10994
    * 参考: 上記のissueについて
        * `--obfuscate`を利用した際にClashlyticsのスタックトレースも難読化されてしまう、というIssueとなる。(8934, 10994のissue)
        * これは原因は`--obfuscate`を利用した際に、dSYMの方も難読化がされてしまったこととなる(124715のissue)  
        * これらのissueは現在、解消済み
            * https://firebase.google.com/support/release-notes/ios#crashlytics_8
* 参考
    * ドキュメントやネットの情報では、シンボルファイルのアップロードについて言及されているが、おそらく古い情報と考えられる。
    * https://firebase.google.com/docs/crashlytics/get-started?platform=flutter&hl=ja#add-sdk
        > Flutter プロジェクトが --split-debug-info フラグ（および必要に応じて --obfuscateフラグ）を使用している場合は、Firebase CLI（v.11.9.0 以降）を使用して Android シンボルをアップロードする必要があります。
    * https://stackoverflow.com/questions/77982483/how-to-automatically-de-obfuscate-stack-trace-in-crashlytics
    
