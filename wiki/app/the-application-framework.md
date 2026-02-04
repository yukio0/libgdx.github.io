---
title: アプリケーションフレームワーク
---
## モジュール
libGDXの中核は、OSとやり取りするための手段を提供するインターフェースとしての6つの[モジュール](/wiki/app/modules-overview)で構成されています。各バックエンドは、これらのインターフェースを実装します。

  * *[Application](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Application.java)*: アプリケーションを実行し、ウィンドウのリサイズなどアプリケーションレベルのイベントをAPI利用側に通知します。ログ出力機能や、メモリ使用量などを問い合わせるメソッドも提供します。
  * *[Files](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Files.java)*: プラットフォームの基盤となるファイルシステムへアクセスできるようにします。独自のファイルハンドル機構の上で、さまざまな種類のファイルの場所を抽象化して扱えるようにします（JavaのFileクラスとは相互運用できません）。
  * *[Input](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Input.java)*: マウス、キーボード、タッチ、加速度センサーなどのユーザー入力をAPI利用側に通知します。ポーリング方式とイベント駆動方式の両方をサポートします。
  * *[Net](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Net.java)*: HTTP/HTTPSによるリソースアクセスをクロスプラットフォームに提供し、TCPのサーバー／クライアントソケットも作成できます。
  * *[Audio](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Audio.java)*: 効果音の再生やストリーミング音楽の再生、そして PCM 音声の入出力のためにオーディオデバイスへ直接アクセスする手段を提供します。
  * *[Graphics](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Graphics.java)*: （利用可能な環境では）OpenGL ES 2.0 を利用できるようにし、ビデオモードの取得／設定など、関連する操作を行えます。

## スタータークラス
プラットフォーム固有のコードとして書く必要があるのは、いわゆる[スタータークラス](/wiki/app/starter-classes-and-configuration)だけです。対象プラットフォームごとに、バックエンドが提供するApplicationインターフェースの実装を生成するコードを書きます。デスクトップの場合、LWJGL3バックエンドを使うと次のようになります。

```java
public class DesktopLauncher {
   public static void main(String[] args) {
      Lwjgl3ApplicationConfiguration config = new Lwjgl3ApplicationConfiguration();
      new Lwjgl3Application(new MyGdxGame(), config);
   }
}
```

Android の場合、対応するスタータークラスは次のようになるでしょう。

```java
public class AndroidStarter extends AndroidApplication {
   public void onCreate(Bundle bundle) {
      super.onCreate(bundle);
      AndroidApplicationConfiguration config = new AndroidApplicationConfiguration();
      initialize(new MyGame(), config);
   }
}
```

これら2つのクラスは通常、デスクトップ用プロジェクトとAndroid用プロジェクトのように、別々のプロジェクトに置かれます。プロジェクト構成（レイアウト）については、[プロジェクトの作成](/wiki/start/project-generation)ページで説明されています。

アプリケーション本体のコードは、[ApplicationListener](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/ApplicationListener.java)インターフェースを実装したクラス（上の例ではMyGame）に書きます。このクラスのインスタンスは、それぞれのバックエンドの Application 実装が持つ初期化メソッドに渡されます（上記参照）。するとアプリケーションは、適切なタイミングで ApplicationListenerの各メソッドを呼び出します（[ライフサイクル](/wiki/app/the-life-cycle)を参照）。

スタータークラスの詳細は[スタータークラスと設定](/wiki/app/starter-classes-and-configuration)のページを参照してください。

## モジュールへのアクセス
先述したモジュールは、[Gdx](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Gdx.java)クラスの静的フィールドからアクセスできます。これは本質的にグローバル変数の集合で、libGDXのどのモジュールにも簡単にアクセスできるようにするものです。一般的には悪いコーディング習慣とみなされがちですが、コードベース内のあらゆる場所で頻繁に使う参照を渡し回す苦労を軽減するために、この仕組みを採用しました。

たとえばオーディオモジュールへアクセスするには、次のように書くだけです。

```java
// 16-bit PCMサンプルを書き込める新しいAudioDeviceを作成する
AudioDevice audioDevice = Gdx.audio.newAudioDevice(44100, false);
```

`Gdx.audio`は、アプリ起動時にApplicationインスタンスによって生成されたバックエンド実装への参照です。他のモジュールも同様にアクセスできます。たとえば`Gdx.app`でApplicationを取得し、`Gdx.files`で Files 実装へアクセスし…といった具合です。
