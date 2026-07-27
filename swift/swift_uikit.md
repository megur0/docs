---
title: "UIKit - Swift"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > [一覧(Swift)](./README.md) > UIKit


# UIKit、Storyboard
* (参考) https://tech.amefure.com/swift-uikit
* UIKitの特徴
    * Storyboard
        * Swiftプロジェクトを作成する際に、インターフェースを「Storyboard」と「SwiftUI」の2つから選択できる。
        * UIKitで開発したい場合は「Storyboard」を選択する。
        * Storyboardは、UIKitの機能ではなくXcodeに組み込まれているUI(ユーザーインターフェース)を表示させるための機能。
    * MVCアーキテクチャ
        * UIKitを使ったアプリ開発では、MVCアーキテクチャに則ったアプリ構造とすることがAppleから推奨されている。


# Interface Builder
* XcodeのInterface Builderエディタを使用すると、コードを1行も記述することなく完全なユーザーインターフェイスを構築できる。
* Interface Builderの機能として提供されているのが、Storyboard、Assistant、Auto Layout、Preview。


# Storyboard
* 開発中のアプリの見た目を確認したり、ドラッグ&ドロップでUIコンポーネント(ボタンなど)を配置したり、画面遷移を線で繋げるだけで実装できたりと、コードを書かずに視覚的に開発できる。
* 「Storyboard」を選択した場合、プロジェクトファイル内に「Main.storyboard」と「LaunchScreen.storyboard」が生成される。
* Storyboardでは、ViewControllerという単位でアプリの画面が管理される。


# UIKitで生成されるファイルの役割
* AppDelegate.swift
    * アプリ全体のライフサイクルを管理するクラスが実装されているファイル。
* SceneDelegate.swift
    * AppDelegate.swiftが担っていたシーンで発生するアプリのライフサイクルを管理する。
* ViewController.swift
    * 画面を管理するためのViewControllerクラスが実装されているファイル。Storyboardのsceneと結びつき、さまざまなビューを表示させるために使用する。
    * UIViewControllerをスーパークラスとする。
* Main.storyboard
    * アプリ内の画面表示を閲覧できる。
    * マウス操作でビューの追加や編集、sceneからsceneへの遷移も実装可能。
* Assets.xcassets
    * アプリ内で使用する画像やカラーなどのアセットを管理するファイル。
    * ドラッグ&ドロップで画像を追加したり、アプリ内で使い回せるカスタムカラーを定義できる。
* LaunchScreen.storyboard
    * スプラッシュ画面と呼ばれる、アプリ起動時に最初に表示される画面を管理するファイル。
* info.plist
    * アプリ内の設定を保持するファイル。
* プロジェクト名.xcodeproj
    * 作成したプロジェクトのIDEを起動するためのファイル。


# UIKitでのUIの実装方法
* 2つの方法がある。
    * Storyboard
    * コードで実装
* コードで実装する方法でも、デフォルトではStoryboardをViewへの紐づけのために使う必要があるが、まったく使わずに開発する方法もある。
    * (参考) https://zenn.dev/kazumalab/articles/76ca59f82c189cf33d43


# @IBOutlet、@IBAction
* IBはInterface Builderの略。
* @IBOutletは変数の宣言に付けられるアトリビュートで、Storyboardに配置されたUIとの関連付けをInterface Builder側で処理できるようにする。
* @IBActionは、関数とStoryboard上のオブジェクトで発生したイベントを関連付けたいときに使うアトリビュート。
* これらをViewControllerで定義し、Storyboard上でマウス操作によってUIと紐づけを行う。
