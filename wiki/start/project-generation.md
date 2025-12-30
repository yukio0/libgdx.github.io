---
title: "プロジェクトの作成"
description: "libGDXのセットアップツールはlibGDX Gradleプロジェクトの設定に関わるすべての手順を処理します。"
redirect_from:
  - /dev/project_generation/
  - /dev/project-generation/
---

最初のプロジェクトを作成し、必要な依存関係をダウンロードするために、libGDXではセットアップツールが用意されています。

{% include setup_flowchart.html current='1' %}

1. libGDXプロジェクトセットアップツール（gdx-liftoff）をダウンロードします。

    <a href="https://github.com/libgdx/gdx-liftoff/releases/latest" class="btn btn--success">gdx-liftoffのダウンロード</a>
2. ダウンロードファイルはリリースのAssetsセクションにあります。拡張子が`.jar`のファイルをダウンロードしてください。

3. ダウンロードしたファイルをダブルクリックします。うまく動作しない場合は、コマンドラインツールを開き、.jarファイルを保存したフォルダに移動し、以下のコマンドを実行してください。<br>`java -jar gdx-liftoff-x.x.x.x.jar` ダウンロードしたバージョンに合わせて「x」を置き換えてください。例: `gdx-liftoff-1.12.1.12.jar`
   <br>Linuxの場合、ファイルを右クリックして「プロパティ」を選び、「権限」タブで「プログラムとして実行を許可」をチェックする必要がある場合があります。

これで、以下のセットアップ画面が開き、プロジェクトを作成できるようになります:

![Setup UI](https://github.com/libgdx/gdx-liftoff/raw/master/.github/screenshot.png){: style="width: 500px;" }

ビデオガイド<a href="https://youtu.be/VF6N_X_oWr0">GDX-Liftoff: libGDX Project Setup</a>を参照するか、以下の手順に沿って進めてください。

## プロジェクト
以下のパラメータを入力するよう求められます:

* **PROJECT NAME**: アプリケーションの名前。文字、数字、アンダースコア、ハイフンを使用できます。例: `YourProjectName`<br>
* **PACKAGE**: コードを配置するJavaパッケージ。例: `io.github.some_example_name`<br>
* **MAIN CLASS**: アプリのメインゲームJavaクラス名。例: `Main`<br>

## アドオン
Project Optionsボタンをクリックすると、アドオン画面に移動します:

* **Platforms**: プロジェクトがサポートするバックエンド。Coreは全プロジェクトで必須です。Desktopはテスト用に強く推奨されます。その他、サポートしたいデバイス向けに追加プラットフォームを選択してください。<br>

**注意:** iOS向けにコンパイルするには、macOSでのみ利用可能なXcodeが必要です！
{: .notice--info}

* **Languages**: Java以外でプロジェクトに含めたい言語（Groovy、Kotlin、Scala）。<br>
* **Extensions**: libGDXの機能を拡張する公式アドオン。
  * **[Ashley](https://github.com/libgdx/ashley)**: 小規模なエンティティフレームワーク。<br>
  * **[Box2dlights](https://github.com/libgdx/box2dlights)**: Box2Dを用いた2Dライティングフレームワーク。OpenGL ES 2.0でレンダリング。<br>
  * **[Ai](https://github.com/libgdx/gdx-ai)**: 人工知能フレームワーク。<br>
  * **[Box2d](/wiki/extensions/physics/box2d)**: 2D物理演算ライブラリ。<br>
  * **[Bullet](/wiki/extensions/physics/bullet/bullet-physics)**: 3D衝突判定および剛体物理ライブラリ。<br>
  * **[Controllers](https://github.com/libgdx/gdx-controllers?tab=readme-ov-file#%EF%B8%8F-game-controller-extension-for-libgdx-version-2)**: コントローラー（例：XBox 360コントローラー）を扱うためのライブラリ。<br>
  * **[FreeType](/wiki/extensions/gdx-freetype)**: スケーラブルフォント。動的にフォントサイズを変更可能。ただし、HTMLプロジェクトには対応していません。<br>
  * **Tools**: 2D/3Dパーティクルエディタ、ビットマップフォント、画像テクスチャパッカーなどを含むツール群。<br>
* **Template**: プロジェクトに含める基本クラスを定義します。<br>

## サードパーティ
次の画面に進むとサードパーティ画面が表示されます。これらは公式のlibGDXメンテナが提供していない追加の拡張機能です:

* **Search**: ライブラリ名やキーワードでリストをフィルタリングできます。<br>
* **Show only selected**: 選択済みライブラリのみを表示。不要なライブラリの選択解除が簡単になります。<br>
* 後で拡張機能を追加したい場合は、[こちら](/wiki/articles/dependency-management-with-gradle#libgdx-extensions)を参照してください。<br>

## 設定
最終画面ではバージョンやその他オプションを設定できます:

* **libGDX Version**: プロジェクトに含めるlibGDXの公式バージョン。最新安定版は `1.12.1`。最新の修正や機能を反映するには `1.12.2-SNAPSHOT` を使用。スナップショットビルドは不安定でAPI互換性に影響する場合があります。<br>
* **Java Version**: プロジェクトをビルドするJavaのバージョン。<br> 
  * `8`がほとんどのプロジェクトで推奨。<br>
  * 古いAndroid端末やiOSをサポートする場合は`7`。<br>
  * `11`はデスクトップとHTMLでサポート。GWT（HTML5）は[一部のJavaライブラリ](https://www.gwtproject.org/doc/latest/DevGuideCodingBasicsCompatibility)のみサポート。<br>
  * `22`のような最新のJavaはデスクトップのみ対応。<br>
* **App Version**: プロジェクト全体で使用するゲームのバージョン番号。[セマンティックバージョニング](https://semver.org/)を参照。<br>
* **Add GUI Assets**: Scene2D UI用の汎用スキンを追加。<br>
* **Add README**: プレースホルダ付きの基本READMEファイルを追加。READMEのTipsを参照してGradleコマンドを学べます。<br>
* **Add Gradle Tasks**: プロジェクト作成後、Gradleコマンドを追加。<br>
* **Project Path**: プロジェクトの出力先フォルダ。<br>
* **Android SDK Path**: Androidをターゲットに選んだ場合、ここにSDKのパスを指定。<br>
  * Linuxデフォルトパス: `~/Android/Sdk`<br>
  * Macデフォルトパス: `~/Library/Android/sdk`<br>
  * Windowsデフォルトパス: `%LOCALAPPDATA%\Android\Sdk`<br>
  * Android Studioでは、ウェルカム画面の「More Actions」→「SDK Manager」で確認可能。<br>

## プロジェクト作成
Generateをクリックすると、プロジェクトのサマリ画面が表示されます:

* ファイル作成中に発生したエラーはスタックトレース付きでここに表示されます。<br>
* IntelliJ IDEAをインストールしていれば、プロジェクトを直接開くことができます。<br>
* 「New Project」をクリックすると、最初からプロジェクト作成を始めることができます。

## プロジェクト構成
プロジェクト作成によって以下のディレクトリ構成が作成されます:

```
gradle.properties          <- プロジェクト全体で使用するバージョン番号などのグローバル変数。
settings.gradle            <- サブモジュールの定義。デフォルトはcore、desktop、android、html、ios。
build.gradle               <- メインGradleビルドファイル。
gradlew                    <- ローカルGradleラッパー。
gradlew.bat                <- Windows上でGradleを実行するスクリプト。
local.properties           <- IntelliJ専用ファイル、Android SDKの場所を定義。

assets/                    <- 画像や音声などのアセットを格納。

core/
    build.gradle           <- coreプロジェクト用Gradleビルドファイル。依存関係を定義。
    src/                   <- ゲームのコードを置くソースフォルダ。

lwjgl3/
    build.gradle           <- デスクトッププロジェクト用Gradleビルドファイル。デスクトップ専用の依存関係を定義。
    src/                   <- デスクトッププロジェクトのソースフォルダでLWJGLランチャークラスを含む。

android/
    build.gradle           <- Androidプロジェクト用Gradleビルドファイル。Android専用の依存関係を定義。
    AndroidManifest.xml    <- Android固有の設定。
    res/                   <- アプリアイコンやその他リソースを格納。
    src/                   <- AndroidプロジェクトのソースフォルダでAndroidランチャークラスを含む。

html/
    build.gradle           <- HTMLプロジェクト用Gradleビルドファイル。GWT専用の依存関係を定義。
    src/                   <- HTMLプロジェクトのソースフォルダ、HTMLランチャークラスとHTML定義を含む。
    webapp/                <- WARテンプレート。生成時にその内容がWARにコピーされる。起動用URLのindexページとweb.xmlを含む。

ios/
    build.gradle           <- iOSプロジェクト用Gradleビルドファイル。iOS専用の依存関係を定義。
    src/                   <- iOSプロジェクトのソースフォルダ、iOSランチャークラスを含む。
```

## Gradleとは？
libGDXプロジェクトは[Gradle](https://gradle.org/)プロジェクトで、依存関係の管理やビルドが格段に容易になります。

Gradleは**依存関係管理**システムであり、サードパーティライブラリを手動でダウンロードせずにプロジェクトに組み込む方法を提供します。必要なのは組み込みたいライブラリ名とバージョンをGradle設定ファイルに指定することだけです。設定ファイルの数行を変更するだけで、ライブラリの追加、削除、バージョン変更が可能です。指定したライブラリは中央リポジトリ（今回の場合、[Maven Central](https://search.maven.org/)）から取得され、プロジェクト外のディレクトリに保存されます。詳細は[wiki](/wiki/articles/dependency-management-with-gradle)を参照してください。
{: .notice--info}

さらに、Gradleは**ビルドシステム**としても機能し、特定のIDEに依存せずにアプリをビルド・パッケージ化できます。これは、IDEが使えないビルドサーバーや継続的インテグレーション環境で特に便利です。ビルドサーバーはビルドシステムに設定を渡すだけで、異なるプラットフォーム向けにアプリをビルドできます。アプリのデプロイについて詳しく知りたい場合は[こちら](/wiki/deployment/deploying-your-application)を参照してください。
{: .notice--info}

**これで、プロジェクトを[IDEにインポートして実行](/wiki/start/import-and-running)する準備が整いました。**
