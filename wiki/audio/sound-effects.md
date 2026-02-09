---
title: 効果音
---
効果音は小さな音声サンプルで、通常は数秒以内の長さです。キャラクターがジャンプしたり銃を撃ったりといった、特定のゲームイベントに合わせて再生されます。

効果音はMP3、OGG、WAVなどさまざまな形式で保存できます。どの形式を使うべきかは用途次第で、各形式にはそれぞれ長所と短所があります。たとえばWAVファイルは他の形式に比べてサイズが大きく、OGGファイルはRoboVM（iOS）やSafari（GWT）では動作しません。またMP3ファイルはシームレスなループ再生に問題があります。

**注意：** Androidでは、`Sound`インスタンスのサイズは1MBを超えられません（ファイルサイズではなく、展開後の非圧縮RAW PCMサイズ基準です）。より大きいファイルを扱う場合は、代わりに[`Music`](/wiki/audio/streaming-music)を使用してください。
{: .notice--primary}

効果音は、[Sound](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/audio/Sound.html)インターフェースで表現されます。効果音の読み込みは次のとおりです。

```java
Sound sound = Gdx.audio.newSound(Gdx.files.internal("data/mysound.mp3"));
```

これは内部ディレクトリ`data`から`"mysound.mp3"`という音声ファイルを読み込みます。

読み込んだら、次のように再生できます。

```java
sound.play(1.0f);
```

これは効果音を1回、最大音量で再生します。1つの`Sound`インスタンスに対して`play()`メソッドは連続して何度でも呼び出せます（例：ゲーム中の連射など）。その場合、音は重なって再生されます。

さらに細かい制御も可能です。`Sound.play()`を呼ぶたびに、その再生インスタンスを識別する`long`値が返ります。このハンドルを使うことで、特定の再生インスタンスだけを操作できます。

```java
long id = sound.play(1.0f); // 新しい音を再生し、後で操作できるようハンドル（ID）を保持する
sound.stop(id);             // その音の再生を即座に停止する
sound.setPitch(id, 2);      // ピッチを元の2倍に上げる

id = sound.play(1.0f);      // もう一度再生する（別インスタンスとして扱われる）
sound.setPan(id, -1, 1);    // 左側にパンを振り、音量は最大にする
sound.setLooping(id, true); // ループ再生を続ける
sound.stop(id);             // ループ再生を停止する
```

***注意：*** これらの変更系メソッドは、現時点ではJavaScript/WebGLバックエンドでは機能が制限されています。1.9.6時点では、`setPan()`はFlashがサポートされていて、かつ有効化されている場合（`GwtApplicationConfiguration.preferFlash = true`）にのみ動作します。

***注意：*** `setPan()`メソッドはステレオ音源では動作しません。

`Sound`が不要になったら、必ず破棄してください。

```java
sound.dispose();
```

破棄後にその`Sound`へアクセスすると、動作は未定義となりエラーの原因になります。

### Androidで複数の音を鳴らすとフリーズする
[オーディオ](/wiki/audio/audio)の項でも述べたとおり、Androidのオーディオには全般的にさまざまな問題があります。その1つとして、サウンドIDの取得待ちにかなり時間がかかる場合があります。libGDXの`Sound`はデフォルトで同期的に再生されるため、同時に大量の音を鳴らすとメインループが目立って長時間止まってしまうことがあります。特にAndroid 10ではこの問題が顕著です。

解決策は、非同期に再生するようにすることです。ただしその場合、IDが必要な`Sound`の各種メソッドを使えなくなります。詳細はここにあります： [オーディオ#Androidでのオーディオ](/wiki/audio/audio#audio-on-android)
