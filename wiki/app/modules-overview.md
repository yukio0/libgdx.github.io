---
title: モジュール概要
---
## はじめに

libGDXは一般的なゲームアーキテクチャの各段階で必要となるサービスを提供する、複数のモジュールから構成されています。

 * *[Input](/wiki/input/input-handling)* - すべてのプラットフォームで統一された入力モデルとハンドラを提供します。利用可能な場合、キーボード、タッチスクリーン、加速度センサー、マウスをサポートします。
 * *[Graphics](/wiki/graphics/graphics)* - ハードウェアが提供するOpenGL ES実装を用いて、画面に画像を描画できるようにします。
 * *[Files](/wiki/file-handling)* - メディアの種類に依存せず、読み書き操作のための便利なメソッドを提供することで、全プラットフォームでのファイルアクセスを抽象化します。
 * *[Audio](/wiki/audio/audio)* - 全プラットフォームでの音声の録音と再生を容易にします。
 * *[Networking](/wiki/networking)* - シンプルなHTTPのGET/POSTリクエストや、TCPのサーバー／クライアントソケット通信など、ネットワーク操作を行うためのメソッドを提供します。

次の図は、シンプルなゲームアーキテクチャにおけるモジュールの位置づけを示しています。

![images/modules-overview.png](/assets/wiki/images/modules-overview.png)

## モジュール

以下では、各モジュールについて、代表的なユースケースを中心に簡単に説明します。より詳しい情報については、それぞれのモジュールに対応するwikiセクションを参照してください！
{: .notice--primary}

### Input
*Input*モジュールは、すべてのプラットフォームでさまざまな入力状態をポーリングできるようにします。
各キー、タッチスクリーン、加速度センサーの状態を取得できます。デスクトップではタッチスクリーンはマウスに置き換わり、加速度センサーは利用できません。

また、イベントベースの入力モデルを使うために、入力プロセッサを登録する手段も提供します。

次のコードは、タッチ（デスクトップではマウス押下）中であれば、現在のタッチ座標を取得します。
```java
if (Gdx.input.isTouched()) {
  System.out.println("Input occurred at x=" + Gdx.input.getX() + ", y=" + Gdx.input.getY());
}
```
同様の方法で、サポートされている各種入力をポーリングして処理できます。

### Graphics
*Graphics*モジュールはGPUとの通信を抽象化し、OpenGL ESのラッパーのインスタンスを取得するための便利なメソッドを提供します。OpenGLインスタンスを取得するために必要な定型（ボイラープレート）コードを肩代わりし、メーカーごとの実装差も吸収します。

基盤となるハードウェアによっては、これらのラッパーが利用できる場合もあれば、利用できない場合もあります。

Graphicsモジュールは、PixmapやTextureを生成するためのメソッドも提供します。

たとえばOpenGL API 2.0のインスタンスを取得するには、次のコードを使います。
```java
GL20 gl = Gdx.graphics.getGL20();
```
このメソッドは、画面へ描画するために利用できるインスタンスを返します。ハードウェア構成がOpenGL ES v2.0をサポートしていない場合はnullが返されます。

次のコードは、画面をクリアして赤で塗りつぶします。
```java
gl.glClearColor(1f, 0.0f, 0.0f, 1);
gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
```
これらのメソッドは常に（lwjgl、jogl、android などの）プラットフォーム固有の実装を返します。そのため、アプリ本体が実装の違いを意識しなくても、対応している限り幅広いプラットフォームで動作します。

サポートされているAPIバージョンは次のとおりです。

| *GLバージョン* |    *アクセス用メソッド*     |
|:------------:|:-------------------------:|
|     2.0      | `Gdx.graphics.getGL20();` |
|     3.0      | `Gdx.graphics.getGL30();` |


Graphicsモジュールについての詳細は、[こちら](/wiki/graphics/graphics)のドキュメントを参照してください。

### Files
*Files*モジュールは、プラットフォームに依存しない汎用的な方法でファイルへアクセスできるようにします。
ファイルの読み書きを簡単に行えます。ただし、書き込みにはプラットフォームのセキュリティ制約に起因するいくつかの制限があります。

Filesモジュールの最も一般的な用途は、すべてのプラットフォームで、アプリケーション配下の同じサブディレクトリからゲームアセット（テクスチャ、サウンドファイルなど）を読み込むことです。
ハイスコアやゲーム状態をファイルへ保存する用途にも非常に役立ちます。

次の例は、$APP_DIR/assets/texturesディレクトリにあるファイルからTextureを作成します。
```java
Texture myTexture = new Texture(Gdx.files.internal("assets/textures/brick.png"));
```
これはAndroidとデスクトップの両方で動作する、とても強力な抽象化レイヤーです。

### Audio
*Audio*モジュールは、音声ファイルの作成と再生を非常に簡単にします。また、サウンドハードウェアへ直接アクセスする手段も提供します。

扱う音声ファイルは2種類で、*Music*と*Sound*があります。どちらもWAV/MP3/OGG形式をサポートします。

Soundインスタンスはメモリ上に読み込まれ、いつでも再生できます。爆発音や銃声のように、ゲーム中で何度も使われる効果音に最適です。

一方、Musicは、ディスク（またはSDカード）上のファイルからのストリーミングです。再生するたびに、ファイルからオーディオデバイスへストリームとして送られます。

次のコードは、ディスク上の*myMusicFile.mp3*を音量50%で繰り返し再生します。
```java
Music music = Gdx.audio.newMusic(Gdx.files.getFileHandle("data/myMusicFile.mp3", FileType.Internal));
music.setVolume(0.5f);
music.play();
music.setLooping(true);
```

### Networking
*Networking*モジュールは、ゲームのネットワーク処理に役立つ機能を提供します。マルチプレイの追加、プレイヤーをあなたのWebサイトへ誘導する処理、その他さまざまなネットワーク関連タスクに利用できます。これらの機能は複数プラットフォームで利用可能ですが、プラットフォームによっては追加の考慮が必要だったり、一部機能が利用できない場合があります。

Networkingモジュールには、低遅延向けに最適化された設定を備えた、構成可能なTCPクライアント／サーバーソケットが含まれています。

また、HTTPリクエストを作成するためのメソッドやユーティリティも用意されています。その1つがRequest Builderで、メソッドチェーンによって簡単にHTTPリクエストを作成できます。

Request Builderを使ってHTTPリクエストを作成する例は次のとおりです。
```java
HttpRequestBuilder requestBuilder = new HttpRequestBuilder();
HttpRequest httpRequest = requestBuilder.newRequest()
   .method(HttpMethods.GET)
   .url("http://www.google.de")
   .build();
Gdx.net.sendHttpRequest(httpRequest, httpResponseListener);
```

次のコードのように、引数（コンテンツ）付きのHTTPリクエストを作成することもできます。
```java
HttpRequestBuilder requestBuilder = new HttpRequestBuilder();
HttpRequest httpRequest = requestBuilder.newRequest()
   .method(HttpMethods.GET)
   .url("http://www.google.de")
   .content("q=libgdx&example=example")
   .build();
Gdx.net.sendHttpRequest(httpRequest, httpResponseListener);
```
