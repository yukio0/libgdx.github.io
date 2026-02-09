---
title: PCMオーディオの録音
---
PCやAndroid端末では、[AudioRecorder](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/audio/AudioRecorder.html)（[コード](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/audio/AudioRecorder.java)）インターフェースを介して、マイクからPCMデータを取得できます。このインターフェースのインスタンスを作成するには、次のようにします。

```java
AudioRecorder recorder = Gdx.audio.newAudioRecorder(22050, true);
```

これは、サンプリングレート22.05kHz・モノラルモードの`AudioRecorder`を作成します。レコーダーを作成できなかった場合は、`GdxRuntimeException`がスローされます。

サンプルは16-bit符号付きPCMとして読み取れます。

```java
short[] shortPCM = new short[1024]; // 1024サンプル
recorder.readSamples(shortPCM, 0, shortPCM.length);
```

ステレオの場合、サンプルは通常どおりインターリーブされます（1つ目のサンプルが左チャンネル、2つ目のサンプルが右チャンネル）。

`AudioRecorder`はネイティブリソースなので、不要になったら破棄する必要があります。

```java
recorder.dispose();
```

JavaScript/WebGLバックエンドでは、音声録音はサポートされていません。
