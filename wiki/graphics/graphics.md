---
title: グラフィックス
---
# 書き直しが必要
このページは書き直しが必要です。

# はじめに

[Graphics](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Graphics.html)モジュールは、現在のデバイスのディスプレイおよびアプリケーションウィンドウに関する情報に加えて、現在のOpenGLコンテキストに関する情報と、そのコンテキストへのアクセス手段を提供します。具体的には、画面サイズ、ピクセル密度、そしてフレームバッファの特性（色深度、深度／ステンシルバッファ、アンチエイリアス対応など）に関する情報をこのクラスから取得できます。ほかの一般的なモジュールと同様に、[Gdxクラス](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Gdx.html)のstaticフィールド経由でアクセスします。

# OpenGLコンテキスト

このモジュールの用途のひとつとして、より低レベルなコマンド実行や問い合わせのために、現在の OpenGLコンテキストへ直接アクセスすることが挙げられます。

次の例では、OpenGL ES2アプリケーションでコンテキストにアクセスし、ビューポート設定と、フレーム／深度バッファのクリアを行っています。

```java
Gdx.gl20.glViewport( 0, 0, Gdx.graphics.getWidth(), Gdx.graphics.getHeight() );
Gdx.gl20.glClearColor( 0, 0, 0, 1 );
Gdx.gl20.glClear( GL20.GL_COLOR_BUFFER_BIT | GL20.GL_DEPTH_BUFFER_BIT );
```

ここでは、ビューポートを設定するために`getWidth()`/`getHeight()`を使って現在のアプリケーションウィンドウの寸法を取得している点、および通常のOpenGLプログラムと同様にGL20クラスの定数を使っている点に注目してください。libGDX の大きな利点は、高レベルの抽象化だけでは足りない場面で、必要に応じて低レベル機能へアクセスできるところです。

OpenGL ESの各バージョンは、それぞれ対応するインターフェースとして提供されます。また、バージョン非依存のコマンド向けにGLCommonインターフェースもある、という扱いになっています（**ただしこれは存在しません。このページは書き直しが必要です！**）。なお、[GL20](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/graphics/GL20.html)を使用するには、起動時にアプリケーションへOpenGL ES2を使うよう指示する必要があります。

OpenGL Utilityクラスへのアクセスも提供されます（**存在しません。このページは書き直しが必要です！**）が、実際には、この手の機能はlibGDX独自の[Orthographic](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/graphics/OrthographicCamera.java)／[Perspective](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/graphics/PerspectiveCamera.java)カメラクラスで扱ったほうがよいでしょう。拡張機能の対応状況を問い合わせる簡単な方法として`supportsExtension()`もあります。拡張機能名を渡すだけで、現在のデバイスでサポートされているか判定できます。

# フレーム時間

Graphicsクラスで特に便利なメソッドのひとつが`getDeltaTime()`です。これは前回フレームを描画してからの経過時間を返します。フレーム非依存（固定タイムステップ等）が不要な場合の、時間ベースのアニメーションに役立ちます。たとえば、[Stage](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/scenes/scene2d/Stage.java)の中で動く [Actor](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/scenes/scene2d/Actor.java)や[UI](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/#gdx%2Fscenes%2Fscene2d%2Fui)のアニメーションは、アプリケーションの`render`メソッド内で次のように制御できます。

```java
stage.act( Math.min( Gdx.graphics.getDeltaTime(), 1/30 ) );
```

ここでは最大タイムステップを1/30秒に制限しています。これは、極端に大きな時間差が発生したときにアニメーションがガクッと飛ぶ（大きなジャークが出る）のを避けるためです。この例が示す通り、`getDeltaTime()`は単純なアニメーションには便利ですが、依然としてフレーム依存です。ゲームロジックや物理シミュレーションのような敏感な処理では、[別のタイミング戦略](https://gafferongames.com/post/fix_your_timestep/)を採用したほうがよい場合があります。

もうひとつ便利なのが`getFramesPerSecond()`で、現在のフレームレートの移動平均を返します。簡単な診断用途には十分ですが、より本格的にプロファイリングしたい場合は[FPSLogger](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/graphics/FPSLogger.java)の利用が推奨されます。

# プラットフォーム差異

デスクトップでは、Graphicsクラスを使ってウィンドウのアイコンやタイトルを設定できます。もちろん、アイコンやタイトルという概念がないプラットフォームでは、これらのメソッドは効果がありません。

`setDisplayMode()`と`setVSync()`は、それぞれ表示モードをフルスクリーン／ウィンドウに切り替え、垂直同期（VSync）を有効／無効にします。これらのメソッドが影響するのは一部プラットフォームに限られる点に注意してください。
