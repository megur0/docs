[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# App Store Connectへのアプリの登録
* SKU(Stock Keeping Unit：在庫管理の単位)
    * ユーザには表示されない内部トラッキング用にアプリに与える一意の ID。
    * アカウントにアプリを追加した後は、SKU を変更することはできない。
    * https://developer.apple.com/jp/help/app-store-connect/reference/app-information/
* アクセス制限
    * https://developer.apple.com/jp/help/app-store-connect/manage-your-team/edit-access-to-apps/
        > ユーザが Apple Developer Program に参加する組織の一員で、かつ「Certificates, Identifiers & Profiles (証明書、ID、プロファイル)」にアクセスできる場合は、自動的にすべての App の情報にアクセスできます。これは、「Certificates, Identifiers & Profiles (証明書、ID、プロファイル)」経由での App へのアクセスは制限ができないことによります。
    * おそらく、すべてのユーザーが下記に該当する場合は、「アクセス制限あり」自体の選択肢が不活性となり選択できない。
        * （公式ドキュメントの記述は見つからなかった）
    
# アーカイブ
* Xcodeで、Product > Archive でアーカイブが生成される。
    * ビルドとアーカイブが行われる。
    * これによって.xcarchiveファイルが作成される
        * 参考: https://stackoverflow.com/questions/8591004/difference-between-ipa-and-xcarchive
            > IPA は、YourApp.app バンドルを含む、圧縮された Payload フォルダーです。.app には、イメージ、plist ファイル、圧縮された nib と実行可能ファイル、CodeSigning リソースなどのすべてのアプリケーション リソースが含まれます。
            > xcarchive にはアプリと dsym ファイルが含まれています。クラッシュ ログを非シンボリック化するには .DSYM が必要です。保存した .xcarchive を右クリックし、パッケージの内容を表示を選択して、その内容を確認します。
* アーカイブをすると、Organizerの画面へ遷移する。


# App Store Connectへアップロード
* (未登録であれば)App Store Connectでアプリを登録しておく
* Window > Organizer > Archives > 対象を選択
    * ここで表示されるバージョンはCFBundleShortVersionStringとなり、括弧内の数字（ビルド番号）がCFBundleIdentifierとなる。
    * ただ、ビルド番号はCFBundleIdentifierの番号がApp Store Connectに既に該当の番号がアップロードされている場合は、CFBundleIdentifierの値は無視されその次の番号が自動的に割当されるようだ。
        * 例えば、既に1, 2, 3とアップロードされている際に、CFBundleIdentifierが3の場合は、ビルド番号は3ではなく4が採用される。
    * CFBundleIdentifierが既にアップロードされている番号と連続している必要はない。
        * 例えば、既に1, 2, 3とアップロードされている際に、10にしてアップロードしても反映される。(自動的に4にされるといったことはなく、10が採用される)
* Distribute App を押下 > 対象を選択。
    * 例えば、TestFlightのみであれば、`TestFlight Internal Only`

# TestFlight
* https://help.apple.com/xcode/mac/current/#/dev2539d985f
* 内部テスト
    * アプリの審査不要
    * App Store Connectに登録されたユーザーのみ。
* 外部テスト
    * アプリの審査が必要
    * App Store Connectに登録不要。
    * ※ ビルドが Xcode や Xcode Cloud から「TestFlight Internal Only」(TestFlight の内部テストのみ) としてアップロードされた場合は、このテストには利用できない。
* (IMO)内部テストでほとんど対応可能で、外部テストを利用することはあまりない？
* スクリーンショットでフィードバックといった機能がある。
## 内部テスト
* (未済の場合)対象のビルドをApp Store Connectへアップロードしておく
* 内部テスターの追加
    * https://developer.apple.com/jp/help/app-store-connect/test-a-beta-version/add-internal-testers/
    * App Store Connectの「ユーザーとアクセス」から追加
    * 役割は Account Holder、Admin、App Manager、Developer、または Marketing である必要がある。
* App Store Connect > 対象のアプリ > Test Flightタブ を開く 
* ビルド > iOS で対象のバージョンを選択して、輸出コンプライアンス等の確認を行う。
* 内部テスト > + でテストグループを登録(自分含め、テストするユーザーを招待)
* ユーザーは届いたメールからTestFlightを開く。

# Ad hoc
* https://developer.apple.com/documentation/xcode/distributing-your-app-to-registered-devices
* TestFlightを利用する以外に、Ad hocで配信する方法がある。
* この方法は、App Store Connectへのアップロードの必要がなく、直接、端末へipaファイルをインストールすることができる。
* ただし、TestFlightと比較して以下のデメリットがある
    * 対象端末はApple Developer上で登録された端末であり、Provisioning Profileに紐づけがされている必要がある。
    * アプリの自動アップデートはされない
## インストール方法
* (未済であれば)アーカイブを作成する。
* Window > Organizer > Archives > 対象を選択
* Distribute App を押下 > Release Testing もしくは Debugging を押下
    * これで ipaファイルがエクスポートされる。
    * ※ 執筆時点(24/7/14)でドキュメントに記載されている「Ad hoc」というメニューはなかった。ただ、上記のどちらのメニューでも動作した。(違いは分からない。)
* 端末へのインストール
    * 方法1
        * Window > Devices and Simulators > 対象の端末を選択 > installed app > + で ipaファイルを選択
        * (IME) 筆者の環境では `The item at (バンドルID) is not a valid bundle.` というエラーが発生してしまい インストールできなかった
    * 方法2
        * 参考
            * https://www.reddit.com/r/iOSProgramming/comments/16m9ho2/cant_install_ipa_files_on_device_after_updating/
        * 対象の端末を接続した状態で、Finderを開き、対象の端末を選択する。
        * そこに、ipaファイルをドラッグ＆ドロップするとインストールされる。
        * なお、既に同じバンドルIDのアプリをインストール済み（ipaファイル、あるいはTestFlight）の場合はD&Dしてもなにも起こらない。


# 輸出コンプライアンス
* https://qiita.com/Sashiiii111/items/3c960b6d9cbb93f9449b
* アメリカやフランスにおいて、暗号化技術を利用するソフトウェアにおいては規制があり、必要に応じて手続きを踏む必要があるため。
* HTTPSなどの標準的な暗号化のみであれば、 「標準的な暗号化アルゴリズム」に該当する。
* 輸出コンプライアンスの確認を都度、表示しないようにする。
    * https://help.apple.com/xcode/mac/current/#/dev0dc15d044
    * ios/Runner/Info.plist に以下を設定。
    ```
    <key>ITSAppUsesNonExemptEncryption</key>
    <false/>
    ```
    * ITSAppUsesNonExemptEncryptionは、"ITSApp は非免除暗号化を使用しています" といった意味のため、「標準的な暗号化アルゴリズムのみ利用」であれば「false（使用していない）」で良い。



# その他
* コンソールへの出力
    * アプリのコンソール出力はエンドユーザーも内容を見ることが可能である。
    * したがって、リリースビルドにおいてコンソールへの出力は必要最低限とすることが良いだあろう。