---
title: 音楽のストリーミング再生
---
数秒を超えるような音声は、RAMにすべて読み込むのではなく、ディスクからストリーミング再生するほうが好ましいです。libGDXでは、それを実現するための`Music`インターフェースが用意されています。

`Music`インスタンスを読み込むには、次のようにします。

```java
Music music = Gdx.audio.newMusic(Gdx.files.internal("data/mymusic.mp3"));
```

これは内部ディレクトリ`data`から`"mymusic.mp3"`というMP3ファイルを読み込みます。

`Music`インスタンスの再生は次のとおりです。

```java
music.play();
```

もちろん、`Music`インスタンスには再生に関するさまざまな属性を設定できます。

```java
music.setVolume(0.5f);                 // 音量を最大の半分に設定する
music.setLooping(true);                // music.stop()が呼ばれるまで繰り返し再生する
music.stop();                          // 再生を停止する
music.pause();                         // 再生を一時停止する
music.play();                          // 再開する
boolean isPlaying = music.isPlaying(); // 再生中かどうか（メソッド名の通り）
boolean isLooping = music.isLooping(); // ループ設定かどうか（これもメソッド名の通り）
float position = music.getPosition();  // 再生位置を秒単位で返す
```

`Music`インスタンスは、バックエンドによっては（Androidなど）重い処理になります。通常、読み込む数は10個程度までに抑え、同時再生も1〜2個までにしておくのがよいでしょう。

`Music`インスタンスが不要になったら、リソースを解放するために破棄する必要があります。

```java
music.dispose();
```
