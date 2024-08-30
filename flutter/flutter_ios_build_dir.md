[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > iOSでビルドされる構成


# FlutterのDebug/Profile/Release と iOSのConfigulations
* Flutterの3つのビルドモードは、iOSのプロジェクトには Project > Info > Configulations として反映されている。
    * これらはproject.pbxprojのXCBuildConfigurationやXCConfigurationListで定義されている
    * XCBuildConfigurationの内容として、ios/Flutter内のxcconfigファイルが読み込みされている。
* Flutterのビルドモード（Debug/Profile/Release）については[こちら](./flutter_arch.md)を参照


# ios/ディレクトリの構成
## ios/Flutter
* Xcodeの設定ファイル(xcconfig, plist)と、Flutterのエンジンのパッケージ（Flutter.podspec）が含まれている。
* xcconfig
    * xcconfigは、Xcode の Build Settings をコードで設定するファイル
    * デフォルトでFlutterのビルドモードに相当するxcconfigファイルが含まれる
        * Release.xcconfig
        * Debug.xcconfig
            * 例(Pods-Runnerはプラグインを利用している場合)
                ```
                #include? "Pods/Target Support Files/Pods-Runner/Pods-Runner.debug.xcconfig"
                #include "Generated.xcconfig"
                ```
        * ※ Profile.xcconfig は含まれていなかった。
    * Generated.xcconfig
        * Release.xcconfigやDebug.xcconfigは共通で「Generated.xcconfig」をincludeしており、ここに主な記述がされている。
        * 例えばDartファイルのエントリーポイントや、バージョン・ビルド番号など
        * 上記に加えて、flutter runのオプションの--dart-defineや--dart-define-from-fileを利用した際に、Xcodeプロジェクトへの橋渡しを行う。
    * CocoaPodsで管理されるライブラリを追加すると、Debug.xcconfigやRelease.xcconfigに以下の行が追加されて`Pods-Runner.debug.xcconfig`を読み込む。
        ```
        #include? "Pods/Target Support Files/Pods-Runner/Pods-Runner.debug.xcconfig"
        ```
    * ビルド時に自動生成されるios/Flutter/Generated.xcconfigを、Debug.xcconfigとRelease.xcconfigがincludeしている。
* AppFrameworkInfo.plist
    * https://docs.flutter.dev/deployment/ios#updating-the-apps-deployment-version
        > If you changed Deployment Target in your Xcode project, open ios/Flutter/AppframeworkInfo.plist in your Flutter app and update the MinimumOSVersion value to match.
    * ほとんど触れることはないが、project.pbxproj(後述)内のビルドフェーズで読み込みされているため、このファイルの設定はベースとなると考えられる。
        * ただ、ほとんどの設定はinfo.plist等によって上書きされるため、このファイル自体を編集する必要はほとんどない。
    * ただ、上記の公式の記述にあるようにDeployment Targetを変更した際にはこのファイルのMinimumOSVersionの項目と整合性をとる必要がある。
* Flutter.podspec
    * s.sourceとして'https://github.com/flutter/engine'が指定されている
    * これがエンジン本体となる。
* flutter_export_environment.sh
    * 各環境変数をexport
    * flutter runのオプションの--dart-defineや--dart-define-from-fileで定義した変数も"DART_DEFINES"としてBase64でエンコードされた状態でexportされる。
## CocoaPods, プラグイン
* Flutterのプラグインをpubspec.ymlへ追加してビルドや、futter pub addを実行すると、ios/.symlinks/pluginsや、cocoapods関係のファイル・フォルダが自動生成される。
* ios/.symlinks/plugins
    * プラグイン関連を追加してビルドするとこのディレクトリが生成される。
    * ここには各プラグインのFlutterプロジェクトがそのまま配置される。
    * 各プラグインのフォルダ/lib
        * Dartコード
    * 各プラグインのフォルダ/ios
        * Classes
            * ネイティブコード(Objective-CやSwift)
        * .podspec
            * Firebase等はCocoaPodsのライブラリに依存している。  
* ios/Podfile
    * このPodfileのスクリプトおよびpubspec.yml, pubspec.lockに基づいて、Podのライブラリ、プラグイン(.symlinks/plugins)の追加およびPodfile.lockの生成が行われると考えられる。
    * project 'Runner'
        ```
        project 'Runner', {
            'Debug' => :debug,
            'Profile' => :release,
            'Release' => :release,
        }
        ```
        * Debug、Build、Releaseへシンボル（Rubyの機能）を入れている。
        * ios/Flutter/debug.xcconfigとrelease.xcconfigのどちらを使うかの切り替え等に利用される??
    * ios/Flutter/Generated.xcconfigが存在することをチェックしている。(存在しない場合はエラー)
    * Flutter SDK内の flutter_tools パッケージに含まれる podhelper スクリプトを読み込んで実行。
        ```
        require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)
        ```
        * flutter_ios_podfile_setupを実行
            ```
            flutter_ios_podfile_setup
            ```
        * target 'Runner'に対して、flutter_install_all_ios_pods　（Podsに含まれるすべてのライブラリを読み込む?）
            ```
            target 'Runner' do
                use_frameworks!
                use_modular_headers!
                flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
            end
            ```
        * 各ターゲットに対して、flutter_additional_ios_build_settingsを実行
            ```
            post_install do |installer|
                installer.pods_project.targets.each do |target|
                    flutter_additional_ios_build_settings(target)
                end
            end
            ```
* ios/Podfile.lock
    * 各Podsのバージョン情報を記述
    * 基本的には、ios/Podsの中のライブラリのバージョン情報が記述される。
    * ただし中を覗いてみると、下記のようにFlutterプロジェクト独自の項目として、「Flutter本体」「Flutterのプラグイン」が含まれていることがわかる。
        * ※ 例としてimage_picker_iosやintegration_test、firebaseをプラグインとして追加している場合となる。
        * image_picker_iosやintegration_testはFlutterのプラグインで`.symlinks/plugins/image_picker_ios/ios`に本体が格納されている事が示されている。
        * 一方、firebase_authも同様だが、加えてこのプラグインはCocoapodsのライブラリである「Firebase/Auth」に依存していることが記述されている。
            * 「Firebase/Auth」はFlutter専用ではなく、iosで使われるFirebaseのauthの為のライブラリとなる。
            * したがって、Flutterはfirebaseのauthの機能をFlutterプラグイン「firebase_auth」を通して「Firebase/Auth」のAPIを呼び出すことで利用していると考えられる。
        ```
        PODS:
            〜
            - Firebase/Auth (10.18.0):
                - Firebase/CoreOnly
                - FirebaseAuth (~> 10.18.0)
            〜
            - firebase_auth (4.14.0):
                - Firebase/Auth (= 10.18.0)
                - firebase_core
                - Flutter
            〜
            - Flutter (1.0.0)
            〜
            - image_picker_ios (0.0.1):
                - Flutter
            - integration_test (0.0.1):
                - Flutter
        ```
        ```
        DEPENDENCIES:
            〜
            - firebase_auth (from `.symlinks/plugins/firebase_auth/ios`)
            〜
            - Flutter (from `Flutter`)
            - image_picker_ios (from `.symlinks/plugins/image_picker_ios/ios`)
            - integration_test (from `.symlinks/plugins/integration_test/ios`)
            〜
        ```
        ```
        EXTERNAL SOURCES:
            〜
            firebase_auth:
                :path: ".symlinks/plugins/firebase_auth/ios"
            〜
            Flutter:
                :path: Flutter
            〜
            image_picker_ios:
                :path: ".symlinks/plugins/image_picker_ios/ios"
            integration_test:
                :path: ".symlinks/plugins/integration_test/ios"
            〜
        ```
* ios/Pods
    * CocoaPodsのライブラリの本体
## Target(ios/Runner)
* FlutterではTargetは"Runnner"という名称で生成される。
    * (参考) デフォルト名「Runner」の名称を変更する
        * https://github.com/flutter/flutter/pull/124533
    * なお、テスト用にRunnnerTestsというTargetも生成される。
* FlutterのアークテクチャにおけるEmbedder/Runnter に該当する。
    * https://docs.flutter.dev/resources/architectural-overview
* 以下の役割を持っていると考えられる。
    * Dart VMとFlutterランタイムのホストとして機能するエンジンを生成
    * FlutterViewControllerを生成し、 FlutterViewControllerはエンジンに接続する。
        * UIKitの 入力イベントを エンジンへ渡す
        * エンジンは Impeller を使用してレンダリングされたフレームを表示する。
* Base.lproj/LaunchScreen.storyboard, Main.storyboard
    * FlutterViewControllerが指定されている。        
* AppDelegate.swift
    * このファイルがエントリポイントになる。
    * Flutter(エンジン)をimport
    * FlutterAppDelegateを継承。 FlutterAppDelegate.applicationを実行。
        * コードは下記のファイルとなる。
        * https://github.com/flutter/engine/blob/main/shell/platform/darwin/ios/framework/Source/FlutterAppDelegate.mm
        * https://github.com/flutter/engine/blob/main/shell/platform/darwin/ios/framework/Source/FlutterViewController.mm
    ```
    import UIKit
    import Flutter
    
    @UIApplicationMain
    @objc class AppDelegate: FlutterAppDelegate {
        override func application(
            _ application: UIApplication,
            didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
        ) -> Bool {
            GeneratedPluginRegistrant.register(with: self) // GeneratedPluginRegistrant.mのregisterを呼び出して
            return super.application(application, didFinishLaunchingWithOptions: launchOptions)
        }
    }
    ```
* GeneratedPluginRegistrant.h、GeneratedPluginRegistrant.m
    * このGeneratedPluginRegistrant.registerはAppDelegate.swiftからコールされる。
    * 各プラグインのios側のコード中にregisterWithRegistrarを記載しておくと、flutter pub addした際にこのregisterメソッドから呼び出しされるようにコードがジェネレートされると考えられる。registerWithRegistrarの中で例えばメソッドチャネル等の初期化設定など実行する。
    * 例(integration_testプラグインの場合)
        ```objectivec
        // ios/.symlinks/plugins/integration_test/ios/Classes/IntegrationTestPlugin.m
        static NSString *const kIntegrationTestPluginChannel = @"plugins.flutter.io/integration_test";
        static NSString *const kMethodTestFinished = @"allTestsFinished";
        static NSString *const kMethodScreenshot = @"captureScreenshot";
        static NSString *const kMethodConvertSurfaceToImage = @"convertFlutterSurfaceToImage";
        static NSString *const kMethodRevertImage = @"revertFlutterImage";
        //...
        + (void)registerWithRegistrar:(NSObject<FlutterPluginRegistrar> *)registrar {
            // メソッドチャネルの設定
            FlutterMethodChannel *channel = [FlutterMethodChannel methodChannelWithName:kIntegrationTestPluginChannel
                                                                    binaryMessenger:registrar.messenger];
            [registrar addMethodCallDelegate:[self instance] channel:channel];
        }

        - (void)handleMethodCall:(FlutterMethodCall *)call result:(FlutterResult)result {
            // メソッドチャネルによって呼ばれたメソッドのハンドリング
            if ([call.method isEqualToString:kMethodTestFinished]) {
                self.testResults = call.arguments[@"results"];
                // ...
            } else if ([call.method isEqualToString:kMethodScreenshot]) {
                // ...
            } else if ([call.method isEqualToString:kMethodConvertSurfaceToImage]
                // ...
            } else {
                // ...
            }
        }
        ```
       
* Runner-Bridging-Header.h
    * 上記のGeneratedPluginRegistrantのファイルを読み込み。
    * このファイルは本体やプラグインのproject.pbxprojから読み込まれている。
* info.plist
## Workspace, Project
* CocoaPodsの利用有無（プラグインの利用有無）に関係なくデフォルトでWorkspaceも生成される。
* 両方とも名前はRunnnerとなる。
* Runner.xcodeproj
    * project.pbxproj
        * 内容は後述
* Runner.xcworkspace
    * contents.xcworkspacedata
        * 下記のようにRunner.xcodeprojのプロジェクトが含まれている。CocoaPodsを利用している場合はPods/Pods.xcodeprojも追加される。
        ```
        <?xml version="1.0" encoding="UTF-8"?>
        <Workspace
            version = "1.0">
            <FileRef
                location = "group:Runner.xcodeproj">
            </FileRef>
            <FileRef
                location = "group:Pods/Pods.xcodeproj">
            </FileRef>
        </Workspace>
        ```

# トラブルシュート
* ImpellerのError表示
    ```
    [ERROR:flutter/shell/platform/darwin/graphics/FlutterDarwinContextMetalImpeller.mm(42)] Using the Impeller rendering backend.
    ```
    * この表示は問題ないので特に対応は不要
    * https://github.com/flutter/flutter/issues/136046
* Flutterで詳細のエラー内容が不明な時はXcodeからのビルドすることで詳細のエラーを確認できる場合がある
    * 例えば Flutterでは下記のようなエラーが表示されるが、
        * `Building a deployable iOS app requires a selected Development Team with a Provisioning Profile.〜`
    * Xcodeでビルドすると下記のエラーが確認できる、といったケースがある。
        * `This operation can fail if the version of the OS on the device is incompatible with the installed version of Xcode.`


# project.pbxprojの内容
* 以下はproject.pbxprojの一部を抜き出したものである。
* 各設定項目やファイルに割り当てされたUUIDは省略している。
## フォルダ構造(PBXGroup)
* PBXGroupにて、Xcodeでプロジェクトを開いた際のディレクトリ構成?が定義されている
    * mainGroup
        * Flutter
		* Runner
		* Products
		* RunnerTests
    * RunnerTests
        * RunnerTests.swift
    * Flutter
        * AppFrameworkInfo.plist 
        * Debug.xcconfig
        * Release.xcconfig
        * Generated.xcconfig
    * Products
        * Runner.app
        * RunnerTests.xctest
    * Runnner
        * Main.storyboard
		* Assets.xcassets
		* LaunchScreen.storyboard
		* Info.plist
		* GeneratedPluginRegistrant.h
		* GeneratedPluginRegistrant.m
		* AppDelegate.swift
		* Runner-Bridging-Header.h
    * Frameworks(CocoaPodsを利用している場合)
        * Pods_Runner.framework
```
/* Begin PBXGroup section */
		/* RunnerTests */ = {
			/* RunnerTests.swift */
		};
        /* CocoaPodsを利用している場合 */
        /* Frameworks */ = {
			isa = PBXGroup;
			children = (
				/* Pods_Runner.framework */,
			);
			name = Frameworks;
			sourceTree = "<group>";
		};
		/* Flutter */ = {
			isa = PBXGroup;
			children = (
				/* AppFrameworkInfo.plist */,
				/* Debug.xcconfig */,
				/* Release.xcconfig */,
				/* Generated.xcconfig */,
			);
			name = Flutter;
			sourceTree = "<group>";
		};
		(mainGroup) = {
			isa = PBXGroup;
			children = (
				/* Flutter */,
				/* Runner */,
				/* Products */,
				/* RunnerTests */,
			);
			sourceTree = "<group>";
		};
		/* Products */ = {
			isa = PBXGroup;
			children = (
				/* Runner.app */,
				/* RunnerTests.xctest */,
			);
			name = Products;
			sourceTree = "<group>";
		};
		/* Runner */ = {
			isa = PBXGroup;
			children = (
				/* Main.storyboard */,
				/* Assets.xcassets */,
				/* LaunchScreen.storyboard */,
				/* Info.plist */,
				/* GeneratedPluginRegistrant.h */,
				/* GeneratedPluginRegistrant.m */,
				/* AppDelegate.swift */,
				/* Runner-Bridging-Header.h */,
			);
			path = Runner;
			sourceTree = "<group>";
		};
/* End PBXGroup section */
```
## Project
* mainGroupとtargetsが指定されている。
```
/* Begin PBXProject section */
		/* Project object */ = {
			isa = PBXProject;
			/* ... */
			compatibilityVersion = "Xcode 9.3";
            /* ... */
            mainGroup = (mainGroup);
			/* ... */
			targets = (
				/* Runner */,
				/* RunnerTests */,
			);
		};
/* End PBXProject section */
```
## ビルドフェーズ
* ビルドフェーズに関する定義を抜き出すと下記のようになる。
* 定義より、TargetのRunnerは以下の6つのビルドフェーズに分けられているようだ。
    * Run Script
        * flutter_tools/bin/xcode_backend.shが実行される
    * Sources
        * 以下が読み込みされる?
        * AppDelegate.swift
        * GeneratedPluginRegistrant.m
    * Frameworks
        * CocoaPodsを利用している場合は、Pods_Runner.framework
        * このファイルが実際にFlutterにおいてどういった用途なのか、分からなかった。
            * https://github.com/flutter/flutter/issues/87753
            * https://stackoverflow.com/questions/60458570/flutter-app-store-connect-upload-issue-error-itms-90171-invalid-bundle-struct
    * Resources
        * 以下が読み込みされる?
        * LaunchScreen.storyboard
		* AppFrameworkInfo.plist
		* Assets.xcassets
	    * Main.storyboard
    * Embed Frameworks
        * これはfilesが空になっていたため、不明。
    * Thin Binary
        * flutter_tools/bin/xcode_backend.shが実行される
```
/* Begin PBXCopyFilesBuildPhase section */
		/* Embed Frameworks */ = {
			/* ... */
            files = () /* 空になっていた。何が入る? */
            /* ... */
		};
/* End PBXCopyFilesBuildPhase section */

/* Begin PBXFrameworksBuildPhase section */
		/* Frameworks */ = {
			/* Pods_Runner.framework in Frameworks */,
            /* CocoaPodsを利用している場合のみ。 */
		};
/* End PBXFrameworksBuildPhase section */

/* Begin PBXResourcesBuildPhase section */
		/* Resources */ = {
			/* ... */
		};
		/* Resources */ = {
			/* ... */
			files = (
				 /* LaunchScreen.storyboard in Resources */,
				 /* AppFrameworkInfo.plist in Resources */,
				 /* Assets.xcassets in Resources */,
				 /* Main.storyboard in Resources */,
			);
			runOnlyForDeploymentPostprocessing = 0;
		};
/* End PBXResourcesBuildPhase section */

/* Begin PBXShellScriptBuildPhase section */
		/* Thin Binary */ = {
			/* ... */
			files = (
			);
			inputPaths = (
				"${TARGET_BUILD_DIR}/${INFOPLIST_PATH}",
			);
			/* ... */
			outputPaths = (
			);
			/* ... */
			shellPath = /bin/sh;
			shellScript = "/bin/sh \"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh\" embed_and_thin";
		};
		/* Run Script */ = {
			/* ... */
			files = (
			);
			inputPaths = (
			);
			name = "Run Script";
			outputPaths = (
			);
			/* ... */
			shellPath = /bin/sh;
			shellScript = "/bin/sh \"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh\" build";
		};
/* End PBXShellScriptBuildPhase section */

/* Begin PBXSourcesBuildPhase section */
		/* Sources */ = {
			/* ... */
			files = (
				/* RunnerTests.swift in Sources */,
			);
            /* ... */
		};
		/* Sources */ = {
			/* ... */
            files = (
				/* AppDelegate.swift in Sources */,
				/* GeneratedPluginRegistrant.m in Sources */,
			);
            /* ... */
		};
/* End PBXSourcesBuildPhase section */

/* Begin PBXNativeTarget section */
		/* RunnerTests */ = {
			/* ... */
            buildPhases = (
				/* Sources */,
				/* Resources */,
			);
		};
		/* Runner */ = {
			/* ... */
            buildPhases = (
				/* Run Script */,
				/* Sources */,
				/* Frameworks */,
				/* Resources */,
				/* Embed Frameworks */,
				/* Thin Binary */,
			);
			/* ... */
		};
/* End PBXNativeTarget section */
```
## buildSettings
* XCBuildConfigurationを確認すると、Debug, Profile, Releaseの3つがそれぞれ3つにさらに分割されている。
    * この3つはXCConfigurationListの中で、 TagetのRunnerTests, Runner, ProjectのRunner の 3つのそれぞれに別で割り当てされている。
```
/* Begin XCConfigurationList section */
		/* Build configuration list for PBXNativeTarget "RunnerTests" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				/* Debug */,
				/* Release */,
				/* Profile */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
		/* Build configuration list for PBXProject "Runner" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				/* Debug */,
				/* Release */,
				/* Profile */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
		/* Build configuration list for PBXNativeTarget "Runner" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				/* Debug */,
				/* Release */,
				/* Profile */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
/* End XCConfigurationList section */

/* Begin XCBuildConfiguration section */
		/* Profile */ = {
            isa = XCBuildConfiguration;
			buildSettings = {
                /* ... */
            };
			name = Profile;
		};
        /* Profile */ = {
			isa = XCBuildConfiguration;
			baseConfigurationReference = /* Release.xcconfig */;
			buildSettings = {
                /* ... */
            }
            name = Profile;
        };
        /* Debug */ = {
			isa = XCBuildConfiguration;
			buildSettings = {
				/* ... */
			};
			name = Debug;
		};
		/* Release */ = {
			isa = XCBuildConfiguration;
			buildSettings = {
				 /* ... */
			};
			name = Release;
		};
		/* Profile */ = {
			isa = XCBuildConfiguration;
			buildSettings = {
				/* ... */
			};
			name = Profile;
		};
		/* Debug */ = {
			isa = XCBuildConfiguration;
			buildSettings = {
				/* ... */
			};
			name = Debug;
		};
		/* Release */ = {
			isa = XCBuildConfiguration;
			buildSettings = {
				/* ... */
			};
			name = Release;
		};
		/* Debug */ = {
			isa = XCBuildConfiguration;
			baseConfigurationReference = /* Debug.xcconfig */;
			buildSettings = {
				/* ... */
			};
			name = Debug;
		};
		/* Release */ = {
			isa = XCBuildConfiguration;
			baseConfigurationReference = /* Release.xcconfig */;
			buildSettings = {
				/* ... */
			};
			name = Release;
		};
/* End XCBuildConfiguration section */
```
## 全体 
```
// !$*UTF8*$!
{
	archiveVersion = 1;
	classes = {
	};
	objectVersion = 54;
	objects = {

/* Begin PBXBuildFile section */
    /* GeneratedPluginRegistrant.m in Sources */
    /* RunnerTests.swift in Sources */ 
    /* AppFrameworkInfo.plist in Resources */
    /* AppDelegate.swift in Sources */ 
    /* Main.storyboard in Resources */
    /* Assets.xcassets in Resources */
    /* LaunchScreen.storyboard in Resources */
    /* Pods_Runner.framework in Frameworks */ /* CocoaPodsを利用している場合 */
/* End PBXBuildFile section */

/* Begin PBXContainerItemProxy section */
		/* ... */
/* End PBXContainerItemProxy section */

/* Begin PBXCopyFilesBuildPhase section */
		/* ... */
/* End PBXCopyFilesBuildPhase section */

/* Begin PBXFileReference section */
        /* GeneratedPluginRegistrant.h */
        /* GeneratedPluginRegistrant.m */
        /* Pods_Runner.framework */ /* CocoaPodsを利用している場合 */
        /* RunnerTests.swift */
        /* RunnerTests.xctest */
        /* AppFrameworkInfo.plist */
        /* Runner-Bridging-Header.h */
        /* AppDelegate.swift */
        /* Debug.xcconfig */
        /* Release.xcconfig */
        /* Generated.xcconfig */
        /* Runner.app */
        /* Base */
        /* Assets.xcassets */
        /* Base */
        /* Info.plist */
/* End PBXFileReference section */

/* Begin PBXFrameworksBuildPhase section */
	    /* ... */
/* End PBXFrameworksBuildPhase section */

/* Begin PBXGroup section */
		/* ... */
/* End PBXGroup section */

/* Begin PBXNativeTarget section */
		/* ... */
/* End PBXNativeTarget section */

/* Begin PBXProject section */
		/* ... */
/* End PBXProject section */

/* Begin PBXResourcesBuildPhase section */
		/* ... */
/* End PBXResourcesBuildPhase section */

/* Begin PBXShellScriptBuildPhase section */
		/* ... */
/* End PBXShellScriptBuildPhase section */

/* Begin PBXSourcesBuildPhase section */
		/* ... */
/* End PBXSourcesBuildPhase section */

/* Begin PBXTargetDependency section */
    /* ... */
/* End PBXTargetDependency section */

/* Begin PBXVariantGroup section */
		/* Main.storyboard */ = {
			/* ... */
		};
		/* LaunchScreen.storyboard */ = {
			isa = PBXVariantGroup;
			/* ... */
		};
/* End PBXVariantGroup section */

/* Begin XCBuildConfiguration section */
		 /* ... */
/* End XCBuildConfiguration section */

/* Begin XCConfigurationList section */
    /* ... */
/* End XCConfigurationList section */
    };
	rootObject = /* Project object */;
}



```