---
title: "プロジェクトのインポートと実行"
description: "libGDXプロジェクトを使用するIDEにインポートするには、いくつかの手順があります。"
redirect_from:
  - /dev/import-and-running/
  - /dev/import_and_running/
---

次に、プロジェクトをIDEにインポートする必要があります。

{% include setup_flowchart.html current='2' %}


# プロジェクトのインポート
gdx-liftoff でプロジェクトを生成した直後であれば、「Open in IntelliJ Idea」というオプションをクリックするだけで、すぐに作業を始められます。そうでない場合は、以下の手順に従ってください:

1. **IntelliJ IDEA または Android Studio**では、`build.gradle`ファイルを開き、「Open as Project」を選択します。

   **Eclipse**では、`File -> Import... -> Gradle -> Existing Gradle Project`を選択します（生成したプロジェクトが *workspace 内に存在しない* こと、また *同名のプロジェクトが workspace に存在しない* ことを確認してください）。

   **NetBeans**では、`File -> Open Project`を選択してください。

2. 初回インポート時に依存関係がまだダウンロードされていない場合は、Gradle プロジェクトを更新してください。

   **IntelliJ IDEA/Android Studio**では、`Reimport all Gradle projects`を開き、左上の`View -> Tool Windows -> Gradle`（循環する矢印）をクリックします。

   **Eclipse**では、プロジェクトを右クリックし、`Gradle -> Refresh Gradle Project`を選択してください。

<br/>

# 実行方法
新しくインポートしたプロジェクトを実行するには、IDEとターゲットプラットフォームによって異なる手順を踏む必要があります。
## デスクトップ
### IDEA/Android Studio:
1. 画面右側のGradleタブを開く。<br/>
2. プロジェクトのタスクを展開し、`lwjgl3 -> Tasks -> application -> run`を選択してください。<br/>
  ![](/assets/images/dev/idea/3.png)

   **Android Studio 4.2**では、タスクがデフォルトでは表示されなくなりました。`Settings -> Experimental`から`Configure all Gradle tasks during Gradle Sync`にチェックを入れてください。その後、`File -> Sync Project with Gradle Files`からプロジェクトを同期します。:<br/>
  ![](/assets/images/dev/idea/4.png)
   {: .notice--primary}

<b>別の方法として</b>、実行構成を作成することも出来ます:
1. Lwjgl3Launcherクラスを右クリックしてください。
2. Run Lwjgl3Launcher.main()を選択してください。ただし、アセットが未設定のため失敗します。そのため、アセットフォルダをまず設定します。:<br/>
  ![](/assets/images/dev/idea/5.png)
3. 実行構成（Run Configuration）を開いてください:<br/>
  ![](/assets/images/dev/idea/0.png)
4. lwjgl3プロジェクトを実行した際に自動作成された実行構成を編集し、作業ディレクトリ（Working Directory）を`assets`フォルダに設定してください。:<br/>
  ![](/assets/images/dev/idea/1.png)

    **macOS**では、LWJGL3 プロジェクトに 追加の手順が 1 つ必要です。次のいずれかを行ってください。実行構成のVM Optionsに`-XstartOnFirstThread`を設定する、もしくは`main()`メソッドの先頭に次の実験的なコード行を追加してください。`Lwjgl3ApplicationConfiguration.useGlfwAsync();`これらについての追加情報は、[こちら](/news/2021/07/devlog-7-lwjgl3#do-i-need-to-do-anything-else)にあります。
    {: .notice--warning}
5. 実行ボタンを押して、アプリを実行してください。

### Eclipse:

1. `Gradle Tasks`から`projectname-lwjgl3 -> application -> run`をダブルクリックします。
  ![](/assets/images/dev/eclipse/4.png)

    ウィンドウが表示されない場合、`Window -> Show View -> Other -> Gradle -> Gradle Tasks`から表示してください。
    {: .notice--warning}

<b>別の方法として</b>、実行構成を作成することも出来ます:
1. lwjgl3プロジェクトを右クリックし、`Run as -> Run Configurations...`をクリックしてください。
2. 右側のJava Applicationを選択してください。<br/>
  ![](/assets/images/dev/eclipse/3.png)
3. 左上のアイコンをクリックして、新しい実行構成を作成します。
  ![](/assets/images/dev/eclipse/0.png)
4. `Lwjgl3Launcher`クラスをメインクラスとして指定してください。
5. Argumentsタブをクリックしてください。
6. 下部にあるにある'Working directory'で'Other' -> Workspace...を選択してください。
  ![](/assets/images/dev/eclipse/1.png)

   **macOS**では、LWJGL3 プロジェクトに 追加の手順が 1 つ必要です。次のいずれかを行ってください。実行構成のVM Optionsに`-XstartOnFirstThread`を設定する、もしくは`main()`メソッドの先頭に次の実験的なコード行を追加してください。`Lwjgl3ApplicationConfiguration.useGlfwAsync();`これらについての追加情報は、[こちら](/news/2021/07/devlog-7-lwjgl3#do-i-need-to-do-anything-else)にあります。
   {: .notice--warning}

7. その後、`assets`内にあるアセットフォルダを選択してください。

### NetBeans:
lwjgl3プロジェクトを右クリックし、Runをクリックしてください。

<br/>

## Android
- **IDEA/Android Studio:** AndroidLauncherクラスを右クリックして、Run AndroidLauncherを選択してください。。
- **Eclipse:** Androidプロジェクトを右クリックして、Run As -> AndroidApplicationを選択してください。。
- **NetBeans:** Androidプロジェクトを右クリックして、Run As -> AndroidApplicationを選択してください。

<br/>

## iOS
### IDEA/Android Studio
1. 実行構成を開いてください。
2. RoboVM iOSアプリケーション用の新しい実行構成を作成します。

    ![](/assets/images/dev/idea/2.png)

3. プロビジョニングプロファイルと、シミュレータまたはデバイスのターゲットを選択してください。

   注意: arm64シミュレータはデフォルトでは動作しません。x86_64を使用するか、MetalANGLE RoboVMバックエンド("com.badlogicgames.gdx:gdx-backend-robovm-metalangle:$gdxVersion")を使用してください。
   {: .notice--warning}
4. 作成した実行構成を実行してください。

RoboVM IntelliJ IDEAプラグインの使用方法や設定についての詳細は、[公式ドキュメント](https://mobivm.github.io)を参照してください。

### Eclipse
- iOS RoboVMプロジェクトを右クリックして、**Run As** → 使用したいRoboVMランナーを選択してください。

![](/assets/images/dev/eclipse/2.png)

RoboVM IntelliJ IDEAプラグインの使用方法や設定についての詳細は、[公式ドキュメント](https://mobivm.github.io)を参照してください。

<br/>

## HTML
HTMLターゲットは、コマンドラインから実行するのが最適です。GWTに慣れている場合は、任意のIDEで手動設定しても構いませんが、推奨される方法は ターミナル（またはコマンドプロンプト）を使うことです。

HTMLターゲットは**Super Dev モード** で実行でき、実行中に再コンパイルが可能で、ブラウザ上でデバッグも行えます。

お気に入りのシェルまたはターミナルを開き、プロジェクトディレクトリに移動し、対応するGradleタスクを実行してください。

```
./gradlew html:superDev
```

**Unixの場合** 「permission denied」エラーが出る場合は、次のコマンドでgradlewファイルに実行権限を付与してください。`chmod +x gradlew`
{: .notice--primary}

たくさんのテキストが高速で流れていくのが見えるはずです。すべてが順調に進めば、最後に次の行が表示されるはずです。

![](/assets/images/dev/html/0.png)

その後、[`http://localhost:8080/index.html`](http://localhost:8080/index.html)にアクセスすると、再コンパイルボタン付きでアプリケーションが起動します。

Super Devの設定やデバッグ方法についての詳細は、[GWTドキュメント](http://www.gwtproject.org/articles/superdevmode.html)を参照してください。

<br/>

## コマンドライン
すべてのターゲットは、コマンドラインインターフェースから実行およびデプロイできます。

**デスクトップ:**
```
./gradlew lwjgl3:run
```

**Android:**
```
./gradlew android:installDebug android:run
```

Android用のコマンドライン操作を行う前に、`ANDROID_HOME`環境変数に有効なAndroid SDKを指定する必要があります。Windowsでは、`set ANDROID_HOME=​C:/Path/To/Your/Android/Sdk`コマンドを実行してください。Linuxまたは、macOSでは`export ANDROID_HOME=​/Path/To/Your/Android/Sdk`コマンドを実行してください。または、`sdk.dir /Path/To/Your/Android/Sdk`という内容を含む「local.properties」ファイルを作成する方法もあります。

**iOS:**
```
./gradlew ios:launchIPhoneSimulator
```

**HTML:**
```
./gradlew html:superDev
```

その後、[`http://localhost:8080/index.html`](http://localhost:8080/index.html)にアクセスしてください。

### Gradleタスクが失敗している？
Gradle を実行した際に、ビルドやリフレッシュが失敗して十分な情報が得られない場合は、同じコマンドに`--debug`パラメータを付けて再度実行してください。例：

```
./gradlew lwjgl3:run --debug
```

これによりスタックトレースが表示され、Gradleが失敗している原因をより詳しく確認できます。


<br/>

# 次は何をする？
準備が完了したので、いよいよ本格的なコーディングに取り掛かりましょう。ステップバイステップで学べるガイド[シンプルなゲーム](/wiki/start/a-simple-game)を参照してください。
