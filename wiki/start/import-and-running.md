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
- **IDEA/Android Studio:** Right-click AndroidLauncher -> Run AndroidLauncher
- **Eclipse:** Right-click Android project -> Run As -> AndroidApplication
- **NetBeans:** Right-click Android project -> Run As -> AndroidApplication

<br/>

## iOS
### In IDEA/Android Studio
1. Open Run/Debug Configurations
2. Create a new run configuration for a RoboVM iOS application

    ![](/assets/images/dev/idea/2.png)

3. Select the provisioning profile and simulator/device target

   Note: arm64 simulators are not working by default. Either use x86_64 or use the MetalANGLE RoboVM backend instead ("com.badlogicgames.gdx:gdx-backend-robovm-metalangle:$gdxVersion")
   {: .notice--warning}
4. Run the created run configuration

For more information on using and configuring the RoboVM IntelliJ IDEA plugin please see the [documentation](https://mobivm.github.io).

### In Eclipse
- Right-click the iOS RoboVM project > Run As > RoboVM runner of your choice

![](/assets/images/dev/eclipse/2.png)

For more information on using and configuring the RoboVM IntelliJ IDEA plugin please see the [documentation](https://mobivm.github.io).

<br/>

## HTML
HTML is best suited to be run on command line. You are welcome to manually setup GWT in the IDE of your choice if you are familiar with it, but the recommended way is to drop down to terminal or command prompt.

The HTML target can be run in **Super Dev** mode, which allows you to recompile on the fly, and debug your application in browser.

To do so, open up your favourite shell or terminal, change directory to the project directory and invoke the respective gradle task:

```
./gradlew html:superDev
```

**On Unix:** If you get a permission denied error, set the execution flag on the gradlew file: `chmod +x gradlew`
{: .notice--primary}

You should see lots of text wizzing by, and if all goes well you should see the following line at the end:

![](/assets/images/dev/html/0.png)

You can then go to [`http://localhost:8080/index.html`](http://localhost:8080/index.html), to see your application running, with a recompile button.

For further info on configuring and debugging with SuperDev check the [GWT documentation](http://www.gwtproject.org/articles/superdevmode.html).

<br/>

## Command Line
All the targets can be run and deployed to via the command line interface.

**Desktop:**
```
./gradlew lwjgl3:run
```

**Android:**
```
./gradlew android:installDebug android:run
```

The `ANDROID_HOME` environment variable needs to be pointing to a valid android SDK before you can do any command line wizardry for Android. On Windows, use: `set ANDROID_HOME=​C:/Path/To/Your/Android/Sdk`; on Linux and macOS: `export ANDROID_HOME=​/Path/To/Your/Android/Sdk`. Alternatively you can create a file called "local.properties" with the following content: `sdk.dir /Path/To/Your/Android/Sdk`.

**iOS:**
```
./gradlew ios:launchIPhoneSimulator
```

**HTML:**
```
./gradlew html:superDev
```

Then go to [`http://localhost:8080/index.html`](http://localhost:8080/index.html).

### Gradle tasks are failing?
If whenever you invoke Gradle, the build or refresh fails to get more information, run the same command again and add the `--debug` parameter to the command, e.g.:

```
./gradlew lwjgl3:run --debug
```

This will provide you with a stacktrace and give you a better idea of why gradle is failing.


<br/>

# What to do next?
Now that you're done with the set-up, you can get to do some real coding. Take a look at our post [A Simple Game](/wiki/start/a-simple-game) for a step-by-step guide.
