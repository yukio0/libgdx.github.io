---
title: PCMオーディオの再生
---
オーディオモジュールは、[PCMサンプル](https://en.wikipedia.org/wiki/Pulse-code_modulation)をオーディオハードウェアへ書き込むための、直接アクセスする手段を提供します。

オーディオハードウェアは、[AudioDevice](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/audio/AudioDevice.html) [（ソース）](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/audio/AudioDevice.java)インターフェースによって抽象化されています。

新しい`AudioDevice`インスタンスを作成するには、次のようにします。

```java
AudioDevice device = Gdx.audio.newAudioDevice(44100, true);
```

これは、サンプリング周波数44.1kHzでモノラル出力の新しい`AudioDevice`を作成します。デバイスを作成できなかった場合は、`GdxRuntimeException`がスローされます。

デバイスには、16-bit符号付きPCMまたは32-bit float PCMのデータを書き込めます。

```java
float[] floatPCM = ... 例えば、正弦波（sine wave）から生成 ...
device.writeSamples(floatPCM, 0, floatPCM.length);

short[] shortPCM = ... デコーダから生成 ...
device.writeSamples(shortPCM, 0, shortPCM.length);
```

ステレオを使用する場合、左右チャンネルのサンプルは通常どおりインターリーブされます（1つ目のfloat/shortが左、2つ目のfloat/shortが右）。

レイテンシの指標は次のように問い合わせられます。

```java
int latencyInSamples = device.getLatency();
```

これはオーディオバッファのサイズ（サンプル数）を返すため、レイテンシを把握するための良い指標になります。戻り値が大きいほど、書き込んでから実際に再生（受け取り側に到達）されるまでの時間が長くなります。

なお、ほとんどのAndroid端末ではレイテンシが非常に大きい点に注意してください。リアルタイム音声アプリケーションが実用的な10〜30msの範囲に収めるのは難しく、通常は100ms程度が限界で、多くの端末では最大400ms程度になることもあります。残念ながらこれはドライバ／OSに起因する問題で、回避策はありません。

`AudioDevice`はネイティブリソースなので、不要になったら破棄する必要があります。

```java
device.dispose();
```

JavaScript/WebGLバックエンドでは、PCMの直接出力はサポートされていません。
