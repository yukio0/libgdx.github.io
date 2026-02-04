---
title: モジュール概要
---
## はじめに

libGDXは一般的なゲームアーキテクチャの各段階で必要となるサービスを提供する、複数のモジュールから構成されています。

 * *[Input](/wiki/input/input-handling)* - すべてのプラットフォームで統一された入力モデルとハンドラを提供します。利用可能な場合、キーボード、タッチスクリーン、加速度センサー、マウスをサポートします。
 * *[Graphics](/wiki/graphics/graphics)* - ハードウェアが提供するOpenGL ES実装を用いて、画面に画像を描画できるようにします。
 * *[Files](/wiki/file-handling)* - メディアの種類に依存せず、読み書き操作のための便利なメソッドを提供することで、全プラットフォームでのファイルアクセスを抽象化して扱えるようにします。
 * *[Audio](/wiki/audio/audio)* - 全プラットフォームでの音声の録音と再生を容易にします。
 * *[Networking](/wiki/networking)* - シンプルなHTTPのGET/POSTリクエストや、TCPのサーバー／クライアントソケット通信など、ネットワーク操作を行うためのメソッドを提供します。

次の図は、シンプルなゲームアーキテクチャにおけるモジュールの位置づけを示しています。

![images/modules-overview.png](/assets/wiki/images/modules-overview.png)

## Modules

The following part briefly describes each module providing the most common use cases for each. For more in-depth information, be sure to check out the individual wiki sections for the respective modules!
{: .notice--primary}

### Input
The _Input_ module enables the polling of different input states on every platform.
It allows polling the state of each key, touchscreen and accelerometer. On the desktop the touchscreen is replaced by the mouse while the accelerometer is not available.

It also offers the means to register input processors to use an event based input model.

The following code snippet gets the current touch coordinates if a touch (or mouse down on desktop) event is in progress:
```java
if (Gdx.input.isTouched()) {
  System.out.println("Input occurred at x=" + Gdx.input.getX() + ", y=" + Gdx.input.getY());
}
```
In similar fashion all the supported input means can be polled and handled.

### Graphics
The _Graphics_ module abstracts the communication with the GPU and provides convenience methods to obtain instances of OpenGL ES wrappers. It takes care of all the boilerplate code needed to get hold of the OpenGL instance and handles all implementations provided by the manufacturer.

Depending on the underlying hardware, the wrappers may be or may not be available.

The Graphics module also provides methods to generate Pixmaps and Textures.

For example, to obtain the OpenGL API 2.0 instance, the following code will be used:
```java
GL20 gl = Gdx.graphics.getGL20();
```
The method will return an instance that can be used to draw onto the screen. In case the hardware configuration does not support OpenGL ES v2.0, null is returned.

The following snippet clears the screen and paints it with red.
```java
gl.glClearColor(1f, 0.0f, 0.0f, 1);
gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
```
It always returns the specific implementation of the API (lwjgl, jogl or android), so the main application doesn’t need to know specifics and will work across the whole range of platforms if supported.

The following API versions are supported:

| *GL Version* |    *Method to access*     |
|:------------:|:-------------------------:|
|     2.0      | `Gdx.graphics.getGL20();` |
|     3.0      | `Gdx.graphics.getGL30();` |


To learn more about the Graphics module check its documentation [here](/wiki/graphics/graphics).

### Files
The _Files_ module provides a generic way to access files regardless of the platform.
It makes it easy to read and write files. File writing has some limitations, which are due to the platform security limitations.

The most common use case for the Files module, is to load game assets (textures, sound files) from the same sub-directory of the application for all platforms.
It is also very useful for writing high scores or game state to files.

The following example creates a Texture from a file present in the $APP_DIR/assets/textures directory.
```java
Texture myTexture = new Texture(Gdx.files.internal("assets/textures/brick.png"));
```
This is a very powerful abstraction layer as it works on both Android and desktop.

### Audio
The _Audio_ module makes the creation and playback of audio files extremely simple. It also gives direct access to the sound hardware.

It handles two types of sound files. *Music* and *Sound*. Both types support the WAV, MP3 and OGG formats.

Sound instances are loaded into memory and can be played back any time. It is ideal for in-game sound effects that will be used multiple times, like explosions or gunshots.

Music instances on the other hand are streams from files on the disk (or SD card). Every time a file is played, it is streamed from the file to the audio device.

The following code snippet plays a sound file, _myMusicFile.mp3_, from disk repeatedly with the volume half turned up:
```java
Music music = Gdx.audio.newMusic(Gdx.files.getFileHandle("data/myMusicFile.mp3", FileType.Internal));
music.setVolume(0.5f);
music.play();
music.setLooping(true);
```

### Networking
The _Networking_ module offers functions useful for game networking and can be used to add multiplayer, send players to your website, or perform other networking tasks. These features are available across multiple platforms, although some platforms may require additional considerations or lack certain features.

The Networking module includes configurable TCP client and server sockets with settings optimized for low latency.

There are also methods and utilities for making HTTP requests. One such utility is the Request Builder, which uses method chaining to easily create HTTP Requests.

The Request Builder can be used to create HTTP Requests using the following code snippet:
```java
HttpRequestBuilder requestBuilder = new HttpRequestBuilder();
HttpRequest httpRequest = requestBuilder.newRequest()
   .method(HttpMethods.GET)
   .url("http://www.google.de")
   .build();
Gdx.net.sendHttpRequest(httpRequest, httpResponseListener);
```

It can also be used to create HTTP Requests with arguments using the following code snippet:
```java
HttpRequestBuilder requestBuilder = new HttpRequestBuilder();
HttpRequest httpRequest = requestBuilder.newRequest()
   .method(HttpMethods.GET)
   .url("http://www.google.de")
   .content("q=libgdx&example=example")
   .build();
Gdx.net.sendHttpRequest(httpRequest, httpResponseListener);
```
