---
title: アプリケーションのデプロイ
---
ゲームをデプロイする仕組みは、プラットフォームによって異なります。この記事では、libGDXが公式にサポートしている各プラットフォームへデプロイするために必要なことを整理します。

* [Windows／Linux／Mac OS Xへのデプロイ](#deploy-to-windowslinuxmac-os-x)
* [Androidへのデプロイ](#deploy-to-android)
* [OSへのデプロイ](#deploy-to-ios)
* [Webへデプロイ](#deploy-web)

# Windows／Linux／Mac OS Xへのデプロイ {#deploy-to-windowslinuxmac-os-x}
### JARファイルとして {#as-jar-file}
Windows／Linux／Macにデプロイする最も簡単な方法は、実行可能なJARファイルを作ることです。次のコンソールコマンドで作成できます。
`./gradlew lwjgl3:dist`

`Unsupported class file major version 60`のようなエラーが出る場合、あなたのJavaバージョン（一覧は[こちら](https://stackoverflow.com/q/9170832)参照）が、使用しているGradleのバージョンでサポートされていません。対処として、より古いJDKをインストールしてください。

生成されたJARファイルは`lwjgl3/build/libs/`フォルダに出力されます。このJARには必要なコード一式に加え、`android/assets`フォルダ内のすべてのアセットも含まれます。ダブルクリックで実行するか、コマンドラインから`java -jar jar-file-name.jar`で実行できます。これを動かすには、利用者側にJVMのインストールが必要です。このJARはWindows／Linux／Mac OS Xで動作します！

### 代替（モダンな）デプロイ方法 {#alternative-modern-ways-of-deployment}
JavaアプリをJARファイルとして配布するのは、利用者が適切なJRE（あるいはJRE自体）を入れているとは限らないため、手間がかかったり問題が起きやすかったりします。たとえば次のような方法もあります。

* Javaアプリを配布する非常に便利な方法の1つは、JREを同梱してしまうことです。方法は[このページ](/wiki/deployment/bundling-a-jre)を参照してください。**これはアプリ配布の推奨手段です！**
* Electronを使うことで、HTML5アプリケーションをデスクトップ向けにデプロイできます。詳しくは[こちら](https://medium.com/@bschulte19e/how-to-deploy-a-libgdx-game-with-electron-3f1b37f0c26e)。
* GWTアプリケーションはUWPアプリとしてバンドルすることもできます。詳しくは[こちら](https://web.archive.org/web/20200428040905/https://www.badlogicgames.com/forum/viewtopic.php?f=17&t=14766)。

# Androidへのデプロイ {#deploy-to-android}
`gradlew android:assembleRelease`

これにより、署名されていないAPKファイルが`android/build/outputs/apk`フォルダに生成されます。このAPKをインストールまたは公開する前に、[署名](https://developer.android.com/studio/publish/app-signing)が必要です。上のコマンドで作られるAPKはすでにリリースモードなので、keytoolとjarsignerの手順に従うだけでOKです。さらに、このAPKは、[不明な提供元からのインストール](https://developer.android.com/distribute/marketing-tools/alternative-distribution#unknown-sources)を許可している任意のAndroid端末へインストールできます。

# iOSへのデプロイ {#deploy-to-ios}
*このセクションは、iOSアプリの基本的なデプロイ手順を把握していることを前提とします。*

### 前提条件 {#prerequisites}
App StoreにIPAをアップロードするには、配布用の署名で署名し、プロビジョニングプロファイルに関連付ける必要があります。
プロビジョニングプロファイルと証明書の作成については、Appleの[App Store配布ガイド](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/Introduction/Introduction.html)に従ってください。
それらが用意できたら、ルートの`build.gradle`のiOSプロジェクトで次のように設定します。

```
project(":ios") {
    apply plugin: "java"
    apply plugin: "robovm"

    dependencies {
        // ...
    }

    robovm {
        iosSignIdentity = "[Signing identity name]"
        iosProvisioningProfile = "[provisioning profile name]"
        iosSkipSigning = false
        archs = "thumbv7:arm64"
    }
 }
```

- プロビジョニングプロファイル名は、Developer Portal（プロビジョニングプロファイルを作成した場所）で確認できます。
- 署名ID（Signing identity）名は、キーチェーンの「My Certificates」内で確認できます。

### パッケージング {#packaging}
IPAを作成するには、次を実行します。

`gradlew ios:createIPA`

これにより`ios/build/robovm`フォルダにIPAが生成され、Apple App Storeへ配布できます。
アプリをアップロードするには、Mac App Storeの[Transporter](https://apps.apple.com/us/app/transporter/id1450874784)を使う必要があります。

**注意:** iOS 11以降、iOSプロジェクト内のdataフォルダにアイコンを入れるだけではなく、アセットカタログ（Asset Catalog）を含める必要があります。
含めなくても提出自体はできますが、後から次のようなメッセージを受け取ることがあります。`Missing Info.plist value - A value for the Info.plist key CFBundleIconName is missing in the bundle '...'. Apps that provide icons in the asset catalog must also provide this Info.plist key.`これを修正するには、[アセットカタログを含めるための手順](https://github.com/MobiVM/robovm/wiki/Howto-Create-an-Asset-Catalog-for-XCode-9-Appstore-Submission%3F)に従ってください。

### 追加ガイド {#additional-guides}

iOSへのデプロイは比較的シンプルですが、うまくいかない場合は[こちら](https://medium.com/@bschulte19e/deploying-your-libgdx-game-to-ios-in-2020-4ddce8fff26c)が参考になります。iOSアプリをTestFlightへデプロイしたい場合は、[こちらの記事](https://medium.com/dev-genius/deploying-your-libgdx-game-to-ios-testflight-163cada0696b)も参照してください。

# Webへデプロイ {#deploy-web}
`gradlew html:dist`

これによりアプリがJavaScriptへコンパイルされ、生成されたJavaScript／HTML／アセット一式が`html/build/dist/`フォルダに配置されます。このフォルダの内容は、ApacheやNginxなどのWebサーバで配信する必要があります。通常の静的HTML／JavaScriptサイトと同じように扱ってください。JavaやJava Appletは関与しません！

実行時に`Couldn't find Type for class ...`のようなエラーが出ることがあります。その場合はWikiの[リフレクション](/wiki/utils/reflection)ページを参照し、必要なクラス／パッケージを含めてください。

HTML5/GWT特有の話は、詳しくは[この動画](https://youtu.be/I_85usDvJvQ)も参照してください。

Pythonがインストールされていれば、`html/build/dist`フォルダで次を実行して配布物をテストできます。

**Python 2.x**

`python -m SimpleHTTPServer`

**Python 3.x**

`python -m http.server 8000`

その後、ブラウザで[http://localhost:8000](http://localhost:8000)を開くと、プロジェクトが動作しているのを確認できます。

Node.jsの場合は、`npm install http-server -g`の後に`http-server html/build/dist`を実行し、[http://localhost:8080](http://localhost:8080)へアクセスします。[ドキュメント](https://github.com/indexzero/http-server)

PHPの場合は、`php -S localhost:8000`を実行し、[http://localhost:8080](http://localhost:8080)へアクセスします。[ドキュメント](http://php.net/manual/en/features.commandline.webserver.php)