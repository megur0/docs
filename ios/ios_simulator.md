[TOP(About this memo))](../README.md) > [一覧(iOSアプリ開発)](./README.md) > iOSシミュレータの操作


# iOS simulator
* 拡大縮小
    * option押しながら動かす
* 現在地の設定
    * Features>Location
* ホームボタンのクリック
    * shift + ⌘ + H
* シミュレーターを工場出荷状態（ファクトリリセット）状態にする
    * シミュレーターのメニューバー > Device > Erase All content and settingsをクリック
* app switcher
    * shift + ctl + cmd + H

# キーボード
* I/O > input > Send Keyboard input To Device(opt + cmd + k)
    * キーボードからの入力を有効化
    * (IME)筆者の環境では、有効化しなくてもキーボードから入力は可能だった。ただ、日本語入力が不安定だったがこの項目を有効化することで安定した。
* ソフトウェアキーボード
    * I/O > keyboard > Toggle Software keyboard
    * キーボードから入力するとソフトウェアキーボードは自動的にOFFとなるため注意


# トラブルシュート(IME)
* File > Open Simulator に Simulatorが表示されない。
    * Macを再起動した結果動作した
* シミュレーターで「Unable to boot the Simulator」のエラーが発生して 起動できない。
    * 何回か試してみたり、再起動してうまくいくことがあった。
    * それでも解決しない場合は、システム設定>一般>ストレージ>デベロッパ で以下のデータを削除する。
    * Xcode Caches
    * Project Build Data and Indexes
