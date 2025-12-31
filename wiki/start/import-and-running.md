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

<b>Alternatively</b>, you can create a run configuration:
1. Right-click your Lwjgl3Launcher class
2. Select 'Run Lwjgl3Launcher.main()'. This should fail with missing assets, because we need to hook up the assets folder first:<br/>
  ![](/assets/images/dev/idea/5.png)
3. Open up Run Configurations:<br/>
  ![](/assets/images/dev/idea/0.png)
4. Edit the Run Configuration that was just created by running the lwjgl3 project and set the working directory to point to your `assets` folder:<br/>
  ![](/assets/images/dev/idea/1.png)

    On **macOS**, LWJGL3 projects require one extra step: Either, in your Run Configuration, set the VM Options to `-XstartOnFirstThread`. Or, add the following experimental line to the start of your `main()` method: `Lwjgl3ApplicationConfiguration.useGlfwAsync();` Additional information on this can be found [here](/news/2021/07/devlog-7-lwjgl3#do-i-need-to-do-anything-else).
    {: .notice--warning}
5. Run your application using the run button

### In Eclipse:

1. Double click the `projectname-lwjgl3 -> application -> run` task under `Gradle Tasks`.
  ![](/assets/images/dev/eclipse/4.png)

    If the window is not visible, show it under `Window -> Show View -> Other -> Gradle -> Gradle Tasks` 
    {: .notice--warning}

<b>Alternatively</b>, you can create a run configuration:
1. Right-click your lwjgl3 project -> Run as -> Run Configurations...
2. On the right side, select Java Application: <br/>
  ![](/assets/images/dev/eclipse/3.png)
3. At the top left, click the icon to create a new run configuration:
  ![](/assets/images/dev/eclipse/0.png)
4. As Main class select your `Lwjgl3Launcher` class
5. After that, click on the Arguments tab
6. At the bottom, under 'Working directory' select 'Other' -> Workspace...
  ![](/assets/images/dev/eclipse/1.png)

   On **macOS**, LWJGL3 projects require one extra step: Either, in your Run Configuration, set the VM Options to `-XstartOnFirstThread`. Or, add the following experimental code snippet to your `main()` method: `Lwjgl3ApplicationConfiguration.useGlfwAsync();` Additional information on this can be found [here](/news/2021/07/devlog-7-lwjgl3#do-i-need-to-do-anything-else).
   {: .notice--warning}

7. Then select your asset folder located in `assets`

### In NetBeans:
Right-click the lwjgl3 project -> Run

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
