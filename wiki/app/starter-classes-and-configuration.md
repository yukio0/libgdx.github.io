---
title: スタータークラスと設定
---
- [デスクトップ(LWJGL3)](#desktop-lwjgl3)
- [Android](#android)
  - [ゲームアクティビティ](#game-activity)
  - [ゲームフラグメント](#game-fragment)
  - [マニフェストの設定](#manifest-configuration)
      - [画面の向きと設定変更](#screen-orientation--configuration-changes)
      - [権限](#permissions)
  - [ライブ壁紙](#live-wallpapers)
  - [スクリーンセーバー(aka Daydreams)](#screen-savers-aka-daydreams)
- [iOS/Robovm](#iosrobovm)
- [HTML5/GWT](#html5gwt)
    - [モジュールファイル](#module-files)
    - [GWT固有の注意点](#gwt-specifics)


ターゲットとする各プラットフォームごとに、スタータークラスを書く必要があります。このクラスは、バックエンド固有の`Application`実装と、アプリケーションロジックを実装した`ApplicationListener`を生成します。スタータークラスはプラットフォーム依存なので、各バックエンドでの生成方法と設定方法を見ていきましょう。

この記事は、[プロジェクトの作成](/wiki/start/project-generation)と[プロジェクトのインポートと実行](/wiki/start/import-and-running)の手順に従っており、IDE上でプロジェクトの設定がすでに完了していることを前提とします。
{: .notice--info}

# デスクトップ(LWJGL3) {#desktop-lwjgl3}

libGDX 1.10.1以降、LWJGL3はデスクトップのデフォルトバックエンドになっています。詳しくは[こちら](/news/2021/07/devlog-7-lwjgl3)を参照してください。
{: .notice--info}

`my-gdx-game`内の`Lwjgl3Launcher.java`を開くと、次のようになっています。

```java
package com.me.mygdxgame;

import com.badlogic.gdx.backends.lwjgl3.Lwjgl3Application;
import com.badlogic.gdx.backends.lwjgl3.Lwjgl3ApplicationConfiguration;
import com.me.mygdxgame.MyGdxGame;

public class Lwjgl3Launcher {
    public static void main(String[] args) {
        if (StartupHelper.startNewJvmIfRequired()) return;
        createApplication();
    }

    private static Lwjgl3Application createApplication() {
        return new Lwjgl3Application(new MyGdxGame(), getDefaultConfiguration());
    }

    private static Lwjgl3ApplicationConfiguration getDefaultConfiguration() {
        Lwjgl3ApplicationConfiguration configuration = new Lwjgl3ApplicationConfiguration();
        configuration.setTitle("my-gdx-game");
        configuration.useVsync(true);
        configuration.setForegroundFPS(Lwjgl3ApplicationConfiguration.getDisplayMode().refreshRate);
        configuration.setWindowedMode(640, 480);
        configuration.setWindowIcon("libgdx128.png", "libgdx64.png", "libgdx32.png", "libgdx16.png");
        return configuration;
    }
}
```

まず[Lwjgl3ApplicationConfiguration](https://github.com/libgdx/libgdx/blob/master/backends/gdx-backend-lwjgl3/src/com/badlogic/gdx/backends/lwjgl3/Lwjgl3ApplicationConfiguration.java)のインスタンスを生成します。このクラスでは、初期画面解像度、OpenGL ES 2.0/3.0のどちらを使うか、といった各種設定を指定できます。詳細はこのクラスのJavadocを参照してください。

設定オブジェクトを用意したら、`Lwjgl3Application`を生成します。`MyGdxGame()`クラスが、ゲームロジックを実装するApplicationListenerです。

以降、ウィンドウが作成され、[ライフサイクル](/wiki/app/the-life-cycle)で説明したとおりにApplicationListenerが呼び出されます。

# Android {#android}

## ゲームアクティビティ {#game-activity}

Androidアプリケーションは`main()`メソッドをエントリポイントとして使いません。代わりにアクティビティ（Activity）がエントリポイントになります。`my-gdx-game-android`プロジェクトの`AndroidLauncher.java`を開いてください。

```java
package com.me.mygdxgame;

import android.os.Bundle;

import com.badlogic.gdx.backends.android.AndroidApplication;
import com.badlogic.gdx.backends.android.AndroidApplicationConfiguration;
import com.me.mygdxgame.MyGdxGame;

public class AndroidLauncher extends AndroidApplication {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AndroidApplicationConfiguration configuration = new AndroidApplicationConfiguration();
        configuration.useImmersiveMode = true;
        initialize(new MyGdxGame(), configuration);
    }
}
```

主なエントリポイントはアクティビティの`onCreate()`メソッドです。`AndroidLauncher`は`AndroidApplication`を継承しており、`AndroidApplication`自体は`Activity`を継承している点に注意してください。デスクトップのスタータークラスと同様に、設定インスタンス（[AndroidApplicationConfiguration](https://github.com/libgdx/libgdx/tree/master/backends/gdx-backend-android/src/com/badlogic/gdx/backends/android/AndroidApplicationConfiguration.java)）を作成します。設定後、`AndroidApplication.initialize()`を呼び出し、`ApplicationListener`（`MyGdxGame`）と設定を渡します。利用可能な設定項目については、[AndroidApplicationConfigurationのJavadoc](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/backends/android/AndroidApplicationConfiguration.html)を参照してください。

Androidアプリは複数のアクティビティを持てますが、libGDXゲームは通常単一のアクティビティで構成すべきです。ゲーム内の画面遷移は、別アクティビティとして実装するのではなくlibGDX側（Screenなど）で実装します。というのも、新しいアクティビティを作ることは新しいOpenGLコンテキストを作ることを意味し、時間がかかるうえ、すべてのグラフィックリソースを再読み込みする必要が出てくるためです。

## ゲームフラグメント {#game-fragment}

libGDXゲームは、アクティビティ全体を使う代わりにAndroidの[フラグメント](https://developer.android.com/guide/fragments)（Fragment）上でホストすることもできます。これにより、アクティビティの一部領域に表示したり、レイアウト間で移動させたりできます。libGDX用フラグメントを作るには、`AndroidFragmentApplication`を継承し、`onCreateView()`で次のように初期化します。
```java
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return initializeForView(new MyGdxGame());
    }
```

このコードを動かすには、-androidプロジェクト側でいくつか追加の変更が必要です。
1. まだ追加していない場合は、[AndroidXフラグメントライブラリを-androidプロジェクトとビルドパスに追加](https://developer.android.com/jetpack/androidx/releases/fragment)します。後で `FragmentActivity`を継承するために必要です
2. AndroidLauncherアクティビティが`AndroidApplication`ではなく`FragmentActivity`を継承するように変更します。
3. AndroidLauncherアクティビティに`AndroidFragmentApplication.Callbacks`を実装します。
4. libGDX用フラグメント実装として、`AndroidFragmentApplication`を継承するクラスを作成します。
5. フラグメントの`onCreateView()`に`initializeForView()`のコードを追加します。
6. 最後に、AndroidLauncherアクティビティの内容をlibGDXフラグメントに置き換えます。

例：
```java
// 2. AndroidLauncherアクティビティをAndroidApplicationではなくFragmentActivityを継承するよう変更
// 3. AndroidLauncherアクティビティにAndroidFragmentApplication.Callbacksを実装
public class AndroidLauncher extends FragmentActivity implements AndroidFragmentApplication.Callbacks
{
   @Override
   protected void onCreate (Bundle savedInstanceState)
   {
      super.onCreate(savedInstanceState);

      // 6. 最後にAndroidLauncherアクティビティの内容をlibGDXフラグメントに置き換える
      GameFragment fragment = new GameFragment();
      FragmentTransaction trans = getSupportFragmentManager().beginTransaction();
      trans.replace(android.R.id.content, fragment);
      trans.commit();
   }

   // 4. AndroidFragmentApplicationを継承するクラスを作成（libGDX用フラグメント実装）
   public static class GameFragment extends AndroidFragmentApplication
   {
      // 5. フラグメントのonCreateViewにinitializeForView()を追加
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)
      {  return initializeForView(new MyGdxGame());   }
   }

   @Override
   public void exit() {}
}
```

## マニフェストの設定 {#manifest-configuration}
`AndroidApplicationConfiguration`に加えて、Androidアプリケーションはプロジェクトのルートディレクトリにある`AndroidManifest.xml`によっても設定されます。例は次のとおりです。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
  <uses-feature android:glEsVersion="0x00020000" android:required="true"/>
  <application
      android:allowBackup="true"
      android:fullBackupContent="true"
      android:icon="@drawable/ic_launcher"
      android:isGame="true"
      android:appCategory="game"
      android:label="@string/app_name"
      tools:ignore="UnusedAttribute"
      android:theme="@style/GdxTheme">
    <activity
        android:name="com.me.mygdxgame.android.AndroidLauncher"
        android:label="@string/app_name"
        android:screenOrientation="landscape"
        android:configChanges="keyboard|keyboardHidden|navigation|orientation|screenSize|screenLayout"
        android:exported="true">
        <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>
  </application>

</manifest>
```

#### 画面の向きと設定変更 {#screen-orientation--configuration-changes}
`targetSdkVersion`に加えて、`activity`要素の`screenOrientation`属性と`configChanges`属性は常に設定しておくべきです。

`screenOrientation`はアプリの向きを固定します。アプリが横画面／縦画面のどちらでも動作できるなら、これを省略してもかまいません。

`configChanges`は*非常に重要*で、上記の値を常に指定すべきです。これを省略すると、物理キーボードの出し入れや端末の向きの変更が起きるたびにアプリが再起動されます。`screenOrientation`を省略した場合、向きの変更を知らせるためにlibGDXアプリは`ApplicationListener.resize()`を呼び出します。利用側はそれに応じてレイアウトを再構成できます。

#### 権限 {#permissions}
アプリが端末の外部ストレージ（例：SDカード）へ書き込む必要がある、インターネットアクセスが必要、バイブレーターを使う、音声録音をしたい――といった場合は、次の権限（permission）を`AndroidManifest.xml`に追加する必要があります。

```xml
	<uses-permission android:name="android.permission.RECORD_AUDIO"/>
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	<uses-permission android:name="android.permission.VIBRATE"/>
```

一般に、権限の多いアプリはユーザーに警戒されやすいので、必要なものだけを慎重に選んでください。

ウェイクロック(wake lock)を使うには、`AndroidApplicationConfiguration.useWakeLock`を`true`にする必要があります。

ゲームが加速度センサーやコンパスにアクセスする必要がない場合は、`AndroidApplicationConfiguration`の`useAccelerometer`と`useCompass`を`false`にして無効化することを推奨します。

ゲームでジャイロスコープが必要なら、`AndroidApplicationConfiguration`の`useGyroscope`を`true`に設定してください（省電力のため、デフォルトでは無効です）。

アプリアイコンなど他の属性の設定方法については、[Android Developer's Guide](https://developer.android.com/guide)を参照してください。

## ライブ壁紙 {#live-wallpapers}
libGDXの`core`アプリケーションはAndroidの[ライブ壁紙](https://android-developers.googleblog.com/2010/02/live-wallpapers.html)（Live Wallpapers）として利用することもできます。
プロジェクト構成はAndroidゲームと非常によく似ていますが、`AndroidApplication`の代わりに`AndroidLiveWallpaperService`を使います。ライブ壁紙はアクティビティではなくAndroidの[サービス](https://developer.android.com/guide/components/services)（Service）です。

**注意： 同期の問題があるため、同じアプリ内でゲームとライブ壁紙を組み合わせることはできません。ただし、ライブ壁紙とスクリーンセーバーは同じアプリ内で安全に共存できます。**

まず`AndroidLiveWallpaperService`を継承し、ゲームのアクティビティで行う`onCreate()`の代わりに`onCreateApplication()`をオーバーライドします。

```java
public class MyLiveWallpaper extends AndroidLiveWallpaperService {
    @Override
    public void onCreateApplication() {
        AndroidApplicationConfiguration cfg = new AndroidApplicationConfiguration();

        initialize(new MyGdxGame(), cfg);
    }
}
```

必要に応じて、`ApplicationListener`クラスで`AndroidWallpaperListener`を実装することで、ライブ壁紙特有のイベント通知を受け取るようにできます。`AndroidWallpaperListener`は`core`モジュールからは利用できないため、[プラットフォーム固有コードとの連携](/wiki/app/interfacing-with-platform-specific-code)で示した方法に従うか、あるいは`android`モジュール側だけで管理するために、次のように`ApplicationListener`をサブクラス化して対応できます。

```java
public class MyLiveWallpaper extends AndroidLiveWallpaperService {

    static class MyLiveWallpaperListener extends MyGdxGame implements AndroidWallpaperListener {
        @Override
        public void offsetChange (float xOffset, float yOffset, float xOffsetStep,
                                  float yOffsetStep, int xPixelOffset, int yPixelOffset) {
            // ホーム画面がスクロールされたときに呼ばれる。すべてのランチャーが対応しているわけではない。
        }

        @Override
        public void previewStateChange (boolean isPreview) {
            // 壁紙のプレビュー表示と実行状態の切り替え時に呼ばれる。
        }

        @Override
        public void iconDropped (int x, int y) {
            // ホーム画面にアイコンがドロップされたときに呼ばれる。
        }
    }

    @Override
    public void onCreateApplication() {
        AndroidApplicationConfiguration cfg = new AndroidApplicationConfiguration();

        initialize(new MyLiveWallpaperListener(), cfg);
    }
}
```

libGDX 1.9.12以降、壁紙のドミナントカラーをOSに通知することもできます。Android 8.1以降では、いくつかのAndroidランチャーやロックスクリーンがこれをスタイリングに利用します（例：時計の文字色を変えるなど）。次のようなメソッドを作って色を通知し、[プラットフォーム固有コードとの連携](/wiki/app/interfacing-with-platform-specific-code)で示した方法に従い、`core`モジュールから呼び出せます。

```java
public void notifyColorsChanged (Color primaryColor, Color secondaryColor, Color tertiaryColor) {
    Application app = Gdx.app;
    if (Build.VERSION.SDK_INT >= 27 && app instanceof AndroidLiveWallpaper) {
        ((AndroidLiveWallpaper) app).notifyColorsChanged(primaryColor, secondaryColor, tertiaryColor);
    }
}
```

サービスクラスに加えて、Androidの`res/xml`ディレクトリに`xml`ファイルを作成し、ライブ壁紙のプロパティを定義する必要があります。壁紙選択画面に表示されるサムネイルと説明文、そして任意で設定用アクティビティを指定します。ここではファイル名を`livewallpaper.xml`とします。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wallpaper
    xmlns:android="http://schemas.android.com/apk/res/android"  
    android:thumbnail="@drawable/ic_launcher"
    android:description="@string/description"
    android:settingsActivity="com.mypackage.MyLiveWallpaperSettingsActivity"/>
```

最後に`AndroidManifest.xml`にも追記が必要です。以下は、シンプルな設定アクティビティを持つライブ壁紙の例です。重要な要素は`uses-feature`と`service`ブロックです。`service`に設定したlabelとiconはAndroidのアプリ設定画面に表示されます。設定アクティビティとライブ壁紙サービスの両方は、ライブ壁紙選択画面からアクセスできるよう`exported`を`true`にする必要があります。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.mypackage">
    <uses-feature android:name="android.software.live_wallpaper" />
    <application android:icon="@drawable/icon" android:label="@string/app_name">
        <activity android:name=".MyLiveWallpaperSettingsActivity"
            android:label="@string/app_name"
            android:exported="true" />
        <service android:name=".LiveWallpaper"
            android:label="@string/app_name"
            android:icon="@drawable/icon"
            android:exported="true"
            android:permission="android.permission.BIND_WALLPAPER">
            <intent-filter>
                <action android:name="android.service.wallpaper.WallpaperService" />
            </intent-filter>
            <meta-data android:name="android.service.wallpaper"
                android:resource="@xml/livewallpaper" />
        </service>				  	
    </application>
</manifest>
```

ライブ壁紙にはタッチ入力に関する制限があります。一般に、報告されるポインタは1つだけです。完全なマルチタッチイベントが必要な場合は、`AndroidApplicationConfiguration.getTouchEventsForLiveWallpaper`を`true`に設定してください。

## スクリーンセーバー(aka Daydreams) {#screen-savers-aka-daydreams}
libGDXの`core`アプリケーションは、Androidの[スクリーンセーバー](https://developer.android.com/about/versions/android-4.2#Daydream)として利用することもできます。スクリーンセーバーはかつて Daydreamと呼ばれていたため、関連クラス名の多くに"Daydream"という語が含まれています。なお、スクリーンセーバーはGoogleのDaydream VRプラットフォームとは無関係です。

プロジェクト構成はAndroidゲームと非常によく似ていますが、`AndroidApplication`の代わりに`AndroidDaydream`を使います。スクリーンセーバーはアクティビティではなくAndroidの[サービス](https://developer.android.com/guide/components/services)です。

まず`AndroidDaydream`を継承し、ゲームアクティビティの`onCreate()`の代わりに、`onAttachedToWindow()`をオーバーライドします。ここでは`super`を必ず呼び出す必要があります。また、このメソッドから`setInteractive()`を呼ぶことで、タッチ操作を有効／無効にできます。非インタラクティブなスクリーンセーバーは、画面に触れるとすぐ終了します。

```java
public class MyScreenSaver extends AndroidDaydream {
    @Override
    public void onAttachedToWindow() {
        super.onAttachedToWindow();
        setInteractive(true);

        AndroidApplicationConfiguration cfg = new AndroidApplicationConfiguration();
        initialize(new MyGdxGame(), cfg);
    }
}
```

サービスクラスに加えて、Androidの`res/xml`ディレクトリに`xml`ファイルを作成し、スクリーンセーバーの唯一の設定項目である「任意の設定アクティビティ」を定義する必要があります。ここではファイル名を`screensaver.xml`とします。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dream xmlns:android="http://schemas.android.com/apk/res/android"
    android:settingsActivity="com.badlogic.gdx.tests.android/.MyScreenSaverSettingsActivity" />
```

最後に`AndroidManifest.xml`にも追記が必要です。以下は、シンプルな設定アクティビティを持つスクリーンセーバーの例です（設定アクティビティは任意です）。重要な要素は`service`ブロックです。設定アクティビティとスクリーンセーバーサービスの両方は、スクリーンセーバー選択画面からアクセスできるよう`exported`を`true`にする必要があります。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.mypackage">
    <application android:icon="@drawable/icon" android:label="@string/app_name">
        <activity android:name=".MyScreenSaverSettingsActivity"
            android:label="@string/app_name"
            android:exported="true" />
        <service android:name=".MyScreenSaver"
            android:label="@string/app_name"
            android:icon="@drawable/icon"
            android:exported="true" >
            <intent-filter>
                <action android:name="android.service.dreams.DreamService" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
            <meta-data android:name="android.service.dream"
                android:resource="@xml/screensaver" />
        </service>			  	
    </application>
</manifest>
```

# iOS/Robovm {#iosrobovm}

`my-gdx-game`内の`IOSLauncher.java`を開くと、次のようになっています。

```java
package com.me.mygdxgame.ios;

import org.robovm.apple.foundation.NSAutoreleasePool;
import org.robovm.apple.uikit.UIApplication;

import com.badlogic.gdx.backends.iosrobovm.IOSApplication;
import com.badlogic.gdx.backends.iosrobovm.IOSApplicationConfiguration;
import com.me.mygdxgame.MyGdxGame;

public class IOSLauncher extends IOSApplication.Delegate {
    @Override
    protected IOSApplication createApplication() {
        IOSApplicationConfiguration configuration = new IOSApplicationConfiguration();
        return new IOSApplication(new MyGdxGame(), configuration);
    }

    public static void main(String[] argv) {
        NSAutoreleasePool pool = new NSAutoreleasePool();
        UIApplication.main(argv, null, IOSLauncher.class);
        pool.close();
    }
}
```

iOSデバイスへのデプロイについて詳しくは、[このMediumの記事](https://medium.com/@bschulte19e/deploying-your-libgdx-game-to-ios-in-2020-4ddce8fff26c)を参照してください。

# HTML5/GWT {#html5gwt}
HTML5/GWTアプリケーションの主なエントリポイントは`GwtApplication`です。`my-gdx-game-html5`プロジェクトの`GwtLauncher.java`を開いてください。

```java
package com.me.mygdxgame.gwt;

import com.badlogic.gdx.ApplicationListener;
import com.badlogic.gdx.backends.gwt.GwtApplication;
import com.badlogic.gdx.backends.gwt.GwtApplicationConfiguration;
import com.me.mygdxgame.MyGdxGame;

public class GwtLauncher extends GwtApplication {
    @Override
    public GwtApplicationConfiguration getConfig () {
        GwtApplicationConfiguration cfg = new GwtApplicationConfiguration(true);
        cfg.padVertical = 0;
        cfg.padHorizontal = 0;
        return cfg;
    }

    @Override
    public ApplicationListener createApplicationListener () {
        return new MyGdxGame();
    }
}
```

エントリポイントは`GwtApplication.getConfig()`と`GwtApplication.createApplicationListener()`の2つのメソッドで構成されます。前者はHTML5アプリケーション向けの各種設定を指定する [GwtApplicationConfiguration](https://github.com/libgdx/libgdx/tree/master/backends/gdx-backends-gwt/src/com/badlogic/gdx/backends/gwt/GwtApplicationConfiguration.java)インスタンスを返す必要があります。`GwtApplication.createApplicationListener()`は実行する`ApplicationListener`を返します。

### モジュールファイル {#module-files}
GWTは参照している各jar／プロジェクトに含まれる実際のJavaコードを必要とします。さらに、各jar／プロジェクトには、拡張子が`gwt.xml`のモジュール定義ファイルが1つ必要です。

例となるプロジェクト構成では、html5プロジェクトのモジュールファイルは次のようになっています。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE module PUBLIC "-//Google Inc.//DTD Google Web Toolkit 2.11.0//EN" "https://www.gwtproject.org/doctype/2.11.0/gwt-module.dtd">
<module rename-to="html">
  <source path="" />

  <inherits name="com.badlogic.gdx.backends.gdx_backends_gwt" />
  <inherits name="com.me.mygdxgame.MyGdxGame" />

  <entry-point class="com.me.mygdxgame.gwt.GwtLauncher" />

  <set-configuration-property name="gdx.assetpath" value="../assets" />
  <set-configuration-property name="xsiframe.failIfScriptTag" value="FALSE"/>
  <set-property name="user.agent" value="gecko1_8, safari"/>
  <collapse-property name="user.agent" values="*" />
</module>
```

ここでは、継承する2つのモジュール（`gdx-backends-gwt`と`core`プロジェクト）に加え、エントリポイントクラス（上の`GwtLauncher`）と、html5プロジェクトのルートディレクトリから見た`assets`ディレクトリへの相対パスを指定しています。

`gdx-backend-gwt`のjarと`core`プロジェクトにも同様のモジュールファイルがあり、他の依存関係が指定されています。*モジュールファイルとソースを含まないjar／プロジェクトは使用できません！*

モジュールと依存関係について詳しくは、[GWT Developer Guide](https://developers.google.com/web-toolkit/doc/1.6/DevGuide)を参照してください。

### GWT固有の注意点 {#gwt-specifics}

HTMLバックエンドにはいくつかの注意点があります。より包括的な[HTML Backend Guide](/wiki/html5-backend-and-gwt-specifics#differences-between-gwt-and-desktop-java)を必ず確認してください。
{: .notice--warning}

GWTはさまざまな理由からJavaの**リフレクション**（reflection）をサポートしません。libGDXには内部のエミュレーション層があり、限られた一部の内部クラスについてはリフレクション情報を生成します。つまり、libGDXの[Jsonシリアライズ](/wiki/utils/reading-and-writing-json)を使うと問題に遭遇することがあります。これを解決するには、どのパッケージ／クラスについてリフレクション情報を生成するかを指定します。詳細は[リフレクションのガイド](/wiki/utils/reflection#gwt)を参照してください。

libGDXのHTML5アプリケーションは、`gdx.assetpath`にあるアセットをすべて事前ロードします。この読み込み処理の間、GWTのウィジェットとして実装された**ローディング画面**が表示されます。このローディング画面をカスタマイズしたい場合は、`GwtApplication.getPreloaderCallback()`メソッド（上の例では`GwtLauncher`内）をオーバーライドするだけでできます。例は[こちら](/wiki/html5-backend-and-gwt-specifics#changing-the-load-screen-progress-bar)にあります。
