---
title: オーディオ
---
# はじめに

libGDXは、大きめの音楽データをディスクから直接ストリーミング再生するためのメソッドだけでなく、小さな効果音を再生するためのメソッドも提供しています。さらに、オーディオハードウェアへの読み書きを簡単に行える便利な手段も用意されています。

オーディオ機能へのアクセスはすべて、次の[audioモジュール](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Audio.html)を通して行います。参照は次のとおりです。

```java
Audio audio = Gdx.audio;
```

libGDXは、アプリケーションが一時停止（pause）および再開（resume）したときに、すべてのオーディオ再生を自動的に一時停止／再開してくれます。


# Androidでのオーディオ

libGDXのAndroidバックエンドは、効果音（`Sound`）の再生に`SoundPool` APIを、音楽（`Music`）の再生に`MediaPlayer`を使用します。これらのAPIには、状況によっていくつかの制限や既知の問題があります。
- レイテンシ（遅延）があまり良くなく、リズムゲームのように遅延に敏感なアプリではデフォルト実装は推奨されません。
- 複数の音声を同時に再生すると、端末によっては性能問題が発生することがあります。簡単な対策として（ただし一部のメソッドが未対応になる制限はありますが）、`AndroidLauncher`で`createAudio()`を次のように実装し、代替のAndroid実装である`AsynchronousAndroidAudio`を使用できます。

```java
@Override
public AndroidAudio createAudio(Context context, AndroidApplicationConfiguration config) {
	return new AsynchronousAndroidAudio(context, config);
}
```

一般に、Androidのオーディオは扱いが難しく、ここで挙げたこと以外にも状況依存や端末固有の問題が起こり得ます。

## 代替案

これらの問題の一部を解決する試みとして、Googleは[Oboe](https://github.com/google/oboe)を開発しました。これは[libGDX Oboe](https://github.com/barsoosayque/libgdx-oboe)を利用することで、libGDXプロジェクトでも使用できます。

別の選択肢として、[gdx-miniaudio](https://github.com/rednblackgames/gdx-miniaudio)プロジェクト経由で[MiniAudio](https://miniaud.io/)を使う方法もあります。これは、積極的にメンテナンスされているクロスプラットフォームのオーディオエンジンで、すでにいくつかのlibGDXゲームで本番利用されています。

libGDXのセットアップツールであるgdx-liftoffには、libGDX-Oboeおよびgdx-miniaudioを読み込むためのオプションがあります。これらはgdx-liftoffの「Third-Party」セクションにあります。

