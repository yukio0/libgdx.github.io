---
title: "シンプルなゲーム"
redirect_from:
  - /dev/simple-game/
  - /dev/simple_game/
---

ゲームを作ってみましょう！ゲームデザインは難しいものですが、作業の流れを小さく達成可能な目標に分解すれば、すばらしいものを生み出すことができます。このシンプルなゲームのチュートリアルでは、ゼロから基本的なゲームを作る方法を学びます。ここで身に付けるスキルは、今後のプロジェクトで土台となる重要なものです。[動画チュートリアル](https://youtu.be/aipDYyh1Mlc)を見ることもできますが、コード例についてはここに戻って確認してください。

{% include embed-gwt.html dir='a-simple-game' width="800" height="500" %}

デモを見ると分かるように、空から落ちてくる水滴をバケツで集める、非常に基本的なゲームを作ります。スコアもゴールもありません。ただ体験を楽しんでください！ゲームデザインの工程を分割するために、以下の手順で進めていきます。

- [前提条件](#前提条件)
- [アセットの読み込み](#アセットの読み込み)
- [ゲームのライフサイクル](#ゲームのライフサイクル)
- [レンダリング](#レンダリング)
- [入力操作](#入力操作)
  - [キーボード操作](#キーボード操作)
  - [マウス、タッチ操作](#マウスタッチ操作)
- [ゲームロジック](#ゲームロジック)
- [効果音と音楽](#効果音と音楽)
- [さらに学ぶには](#さらに学ぶには)
- [完全なサンプルコード](#完全なサンプルコード)

## 前提条件
このチュートリアルを始める前に、いくつか準備が必要です。

  * [構築手順](/wiki/start/setup)に従い、JavaとIDEを準備してください。プロジェクトの作成方法と、適切なGradleコマンドでプロジェクトを実行する方法を理解している必要があります。
  * GDX-Liftoffで、次の設定のプロジェクトを作成してください。
    * Project Name: Drop
    * Package: com.badlogic.drop
    * Main Class: Main
  * CoreとDesktopプラットフォームを含めてください。他のプラットフォームを含めることもできますが、このチュートリアルでは扱いません。この段階で他のプラットフォームを選択すると、予期せぬ新たな問題が発生する可能性があります。
  * ApplicationListenerテンプレートを使用してください。

IDEでプロジェクトを開いてください。Javaコードの知識があれば、libGDXの学習がずっと楽になります。ただし、コードの仕組みを最低限しか理解していなくても、このチュートリアルを進めることはできます。

コード例の中では、チュートリアルの各要素を説明するためにコメントを使用します。コードを書く際に、これらのコメントを必ずしもコピーする必要はありません。

```java
// これはコメントです。コンパイラには無視されます。
```

ただし、分かりにくいコードを説明したり、設計の区切りを示すために、自分でコメントを書くことは良い習慣です。

```java
// バケットの移動をビューポートの幅に制限します。
```

import文はJavaプログラミングにおいて重要な要素です。幸い、最近のIDEではコードを入力している途中で自動的に追加されます。例では省略していますが、実際のコードでは必要になることを前提としています。import文は package文の下、ファイルの先頭に記述されます。

```java
import com.badlogic.gdx.ApplicationListener;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input.Keys;
import com.badlogic.gdx.InputProcessor;
import com.badlogic.gdx.audio.Music;
import com.badlogic.gdx.audio.Sound;
```

libGDX には、他のパッケージに存在するクラスと同じ名前のクラスが存在することがあります。例えば、IDEAで`Rectangle`と入力すると、alt+enterを押してimport文を補完するよう促されます。その際は、libGDXの名前を持つクラスを選択してください。多くの場合、パッケージ名は`com.badlogic.gdx`です。

![libGDX Rectangle class](/assets/images/dev/a-simple-game/1.png)

OpenGLベースのゲーム（libGDX で作られたものなど）では、小数は通常`22.5f`のような浮動小数点数で表現されます。「f」を付け忘れるとビルドエラーになることがあります。`float`が`double`（例：`22.5`）より推奨されるのは、メモリ使用量が少なく、多くのハードウェアでサポートされ、OpenGLが通常`float`を期待するためです。IDEAでは、`ctrl+p`を押すことで、メソッドが期待する引数を確認できます。

![method parameters](/assets/images/dev/a-simple-game/2.png)

コード例中に三点リーダー`...`が表示されている場合、説明を簡潔にするために他のコードが省略されていると考えてください。表示されている行の前後関係から、ファイル内の位置を判断してください。どうしても分からなくなった場合は、ページ下部に完全なサンプルコードがあります。

ゲームをテストする前に、デスクトップウィンドウのサイズを設定します。デスクトップ版の設定はすべて LWJGL3Launcherクラスで行います。プロジェクトフォルダ内で次のファイルを探してください。

![Lwjgl3Launcher](/assets/images/dev/a-simple-game/3.png)

```java
...

private static Lwjgl3ApplicationConfiguration getDefaultConfiguration() {
    Lwjgl3ApplicationConfiguration configuration = new Lwjgl3ApplicationConfiguration();
    configuration.setTitle("Drop");
    configuration.useVsync(true);
    configuration.setForegroundFPS(Lwjgl3ApplicationConfiguration.getDisplayMode().refreshRate + 1);
    configuration.setWindowedMode(800, 500); // この行でウィンドウのサイズを変更します。
    configuration.setWindowIcon("libgdx128.png", "libgdx64.png", "libgdx32.png", "libgdx16.png");

    return configuration;
}
```

## アセットの読み込み
libGDXで作る2Dゲームには、プロジェクトを構成する画像、音声などのアセットが必要です。このゲームでは、バケツ、雨粒、背景、水滴が落ちる効果音、音楽が必要になります。とても器用な人であれば、自分で用意することもできますが、簡単のため、このチュートリアル用に最適化されたサンプルをダウンロードすることもできます。

<a href="/assets/downloads/tutorials/simple-game/bucket.png?nomagnify" download="bucket.png">bucket.png</a><br>
<a href="/assets/downloads/tutorials/simple-game/drop.png?nomagnify" download="drop.png">drop.png</a><br>
<a href="/assets/downloads/tutorials/simple-game/background.png?nomagnify" download="background.png">background.png</a><br>
<a href="/assets/downloads/tutorials/simple-game/drop.mp3?nomagnify" download="drop.mp3">drop.mp3</a><br>
<a href="/assets/downloads/tutorials/simple-game/music.mp3?nomagnify" download="music.mp3">music.mp3</a>

これらのファイルをコンピュータに保存するだけでは不十分です。プロジェクト内のassetsフォルダに配置する必要があります。

![assets folder](/assets/images/dev/a-simple-game/4.png)

ここには、libGDXがサポートしているさまざまなバックエンドごとに、多くのフォルダがあります。assetsフォルダは、すべてのバックエンドで共有されます。ここに保存したものは、ゲームと一緒に配布されます。例えば、デスクトップ版ではJARファイル内にこれらが含まれます。ここで生成されるものが、ユーザーがあなたのゲームを遊ぶためのファイルになります。

注意として、libGDXでは、ファイル名の大文字小文字や拡張子が非常に重要です。`drop.png`、`drop.mp3`、`Drop.mp3`はすべて違うものとして扱われます。厄介なのは、リリースビルドを作るまで問題が表面化しないことが多い点です。Windowsのエクスプローラーでは、ファイル拡張子がデフォルトで非表示になるため、常にエディタ内のプロジェクトブラウザを参照してください。

libGDXはコード重視のフレームワークです。使用するすべてのアセットは、ゲーム内で使う前に、コードで読み込む必要があります。アセットの読み込みはゲーム開始時に行う必要があります。CoreプロジェクトのMain.javaを開いてください。このファイルがこのチュートリアルで主に作業するファイルです。

![Main class](/assets/images/dev/a-simple-game/5.png)

変数はファイル上部にある`public class Main`直下に宣言してください。使用するアセットごとに変数が必要です。

```java
public class Main implements ApplicationListener {
    Texture backgroundTexture;
    Texture bucketTexture;
    Texture dropTexture;
    Sound dropSound;
    Music music;
```

ただし、コンストラクタや初期化時にオブジェクトを生成してはいけません。libGDXの初期化前に生成されてしまうため、正しく動作しません。createメソッドに以下のコードを入力してください。

```java
@Override
public void create() {
    backgroundTexture = new Texture("background.png");
    bucketTexture = new Texture("bucket.png");
    dropTexture = new Texture("drop.png");
}
```

libGDXの起動後にアセットをメモリに読み込みます。背景がテクスチャとして読み込まれている点に注目してください。テクスチャはゲームが画像をビデオメモリに保持する仕組みです。実際には、ゲーム内の各要素ごとに異なるテクスチャを用意するのは効率的ではありません。それらすべてを1つの大きなテクスチャにまとめるべきです。詳細はwikiの[TexturePacker](/wiki/tools/texture-packer)を参照してください。今回は説明を簡単にするため、個別のテクスチャを使用します。

同様に、プロジェクト内の雨音音声ファイルを扱うために「Sound」があります。「Sound」はメモリに完全に読み込まれるため、素早く繰り返し再生できます。一方、「Music」はサイズが大きすぎてメモリに完全に保持できません。ファイルからチャンク単位でストリーミングされます。「Sound」と「Music」の明確な区別ルールはありませんが、10秒未満の音声は「Sound」と見なすことができます。

```java
@Override
public void create() {
    ...
    
    dropSound = Gdx.audio.newSound(Gdx.files.internal("drop.mp3"));
    music = Gdx.audio.newMusic(Gdx.files.internal("music.mp3"));
}
```

管理すべきアセットが多くなってきました。そういうときは[アセットマネージャ](/wiki/managing-your-assets)を使うべきです。アセットマネージャもwikiに記載されています。ただし、繰り返しになりますが、これは本チュートリアルの範囲外です。シンプルなままにしましょう。

## ゲームのライフサイクル
これまでの作業はすべてcreateメソッドの中で行っています。createはゲームの実行時に最初に呼び出されるため、アセットをここで読み込むのは理にかなっています。では、他のメソッドは何のためにあるのでしょうか？libGDXの[ライフサイクル](/wiki/app/the-life-cycle)で詳細が説明されています。

`ApplicationListener`インターフェースを実装しているため、`create()`、`render()`、`resize(int width, int height)`、`pause()`、`resume()`、`dispose()`はすべてこのクラスに含まれています。今後、libGDXでゲームを作るために、これらのメソッドは使うこととなります。多くの高度なシステムでは、これらのメソッドを直接使わない形で抽象化されていますが、それでもこれらはコードの基盤として存在し続けます。

今回の例ではコードの大部分は createメソッドと renderメソッド内に記述されます。今回のようなシンプルなアプリではリソースの解放が不要なため、`dispose`メソッドは使用しません。`dispose`メソッドは複数の画面を持つゲームでより重要になります。

## レンダリング
それではレンダリングについて話しましょう。現代のゲームのほとんどは、テクスチャを操作し、それらを画面に描画することで、最終的に目に見える画像（フレーム）を表示しています。

この処理は1秒間に何度も繰り返され、その連続によって動いているように見える錯覚が生まれます。ここでも、まさにそれを行います。まずはボイラープレートコードから始めましょう。ボイラープレートコードとはほとんど変更することなく何度も使い回す定型コードのことです。これから作るすべてのゲームで、このパターンを見ることになるでしょう。新しい変数を宣言しましょう。

```java
public class Main implements ApplicationListener {
    ...

    SpriteBatch spriteBatch;
    FitViewport viewport;
```

これらの変数をcreateメソッド内で初期化します。

```java
@Override
public void create() {
    ...

    spriteBatch = new SpriteBatch();
    viewport = new FitViewport(8, 5);
}
```

ビューポートは、私たちがゲームをどのように見るかを制御します。それはまるで、こちらの世界からゲーム世界を覗くための「窓」のようなものです。ビューポートはこの「窓」の大きさや、画面上での配置を制御します。使用できるビューポートにはさまざまな種類があります。理解しやすいものの一つがFitViewportです。これはウィンドウのサイズがどのように変わっても、ゲーム全体が常に画面内に収まるようにしてくれます。コンストラクタに渡すパラメータは、ゲーム内の単位でどれくらいの大きさのゲーム世界を表示するかを決めるものです。ゲーム世界はウィンドウのサイズに合わせて「フィット」されます。また、各ビューポートにはカメラが備えられており、ゲーム世界のどの部分をどの倍率で表示するかを制御します。詳しくは、wikiの[ビューポートとカメラ](/wiki/graphics/viewports)を参照してください。

resizeメソッド内では、必ずビューポートを更新することを忘れないでください。

```java
@Override
public void resize(int width, int height) {
    viewport.update(width, height, true); // trueを渡すとカメラを画面中央に移動します
}
```

これからコードはより複雑になっていきます。良い習慣として、コードを複数のメソッドに分割しましょう。その準備として、renderメソッド内で使うための新しいメソッドを追加します。

```java
@Override
public void render() {
    // コードを3つのメソッドに整理します
    input();
    logic();
    draw();
}

private void input() {

}

private void logic() {

}

private void draw() {

}
```

まずは `draw()`メソッドに注目します。

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    spriteBatch.end();
}
```

`ScreenUtils.clear(Color.BLACK);`は画面をクリアします。毎フレーム画面をクリアするのは良い習慣です。これを行わないと、奇妙な描画エラーが発生することがあります。使用する色は何色でも構いませんが、ここでは黒を使うことにします。

お気に入りのゲームで、FPS（フレームレート）が低くなる理由を考えたことはありますか？ゲームがカクつく原因は描画されるテクスチャの数や、プレイヤーのグラフィックカードの性能に関係していることがよくあります。これを軽減するための工夫はいくつかあります。その一つが、描画命令をまとめてGPU（画像処理装置）に送ることです。個々のテクスチャを描画する処理はドローコールと呼ばれます。スプライトバッチはlibGDXがこれらのドローコールをまとめて処理するための仕組みです。

`spriteBatch.setProjectionMatrix(viewport.getCamera().combined);`は、ビューポートがスプライトバッチにどのように適用されるかを示しています。これにより、画像が正しい位置に描画されるようになります。

また、beginとendの正しい順番にすることは重要です。スプライトバッチをbeginとendの外で描画してはいけません。そうした場合はエラーメッセージが表示されます。

```java
spriteBatch.begin();
// ここに描画処理を追加します。
spriteBatch.end();
```

では、実際にやってみましょう。ここで指定する座標は、バケツが画面上のどこに描画されるかを決定します。座標の原点は左下にあり、右方向に行くとx座標が増え、上方向に行くとy座標が増えていきます。このゲームでは、ゲーム世界を想像上の単位で表現します。この想像上の単位をメートルと考えるのがいいでしょう。参考までに、バケツの画像は幅100ピクセル、高さ100ピクセルです。簡単にするため、100ピクセルを1メートルと定義することにします。つまり、バケツの大きさは1×1メートルになります。1メートルあたり何ピクセルにするかという比率は、実際には自由に決められます。ただし、ゲーム世界にとって意味が分かりやすい、シンプルな値を選ぶようにしてください。通常、タイルの大きさやプレイヤーキャラクターの身長などになります。ゲームロジックはピクセルの存在を意識するべきではありません。

![Coordinate Plane](/assets/images/dev/a-simple-game/6.png)

バケツを描画するための行を追加しましょう。

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    spriteBatch.draw(bucketTexture, 0, 0, 1, 1); // 幅と高さを1メートルにしてバケツを描画します。

    spriteBatch.end();
}
```

このコードで、画面左下隅にバケツが描画されるはずです。IDEAで適切なGradleコマンドを実行するか、[環境構築ガイド](/wiki/start/setup)に記載されている使用している開発環境向けの手順を実行してゲームを起動してください。すべてがうまくいっていれば、虚無の闇の中に、我らが勇敢で孤独なバケツが佇んでいるのが見えるはずです。

![bucket in void](/assets/images/dev/a-simple-game/7.png)

背景を追加して場面を盛り上げましょう。背景の描画方法は、バケツを描画したときとほぼ同じです。ビューポート全体を覆うように、背景はビューポートの幅と高さで描画します。

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    // 可読性のため、worldWidthとworldHeightをローカル変数にします。
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    spriteBatch.draw(bucketTexture, 0, 0, 1, 1); // バケツを描画します
    spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight); // 背景を描画します

    spriteBatch.end();
}
```

ゲームを実行すると、背景が表示されるはずです。

![background with no bucket](/assets/images/dev/a-simple-game/8.png)

でも、バケツはどこへ行ってしまったのでしょう？描画順について説明する必要があります。描画はコードに書いた順番どおりに連続して行われます。実際には、次のような処理が起きています。

1. 画面がクリアされる。
2. バケツがバックバッファに描画される。
3. 背景がすべての上に描画され、その最終結果が画面に表示される。

この一連の処理は、毎フレーム繰り返されます。この問題を解決するためには、単にドローコールの順番を入れ替えるだけです。

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    // 次の2行を並び替えます
    spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight); // 背景を描画します
    spriteBatch.draw(bucketTexture, 0, 0, 1, 1); // バケツを描画します

    spriteBatch.end();
}
```

![background with bucket](/assets/images/dev/a-simple-game/9.png)

背景とバケツが正しく表示されていることを確認してください。水滴の描画は後ほど生成処理を実装したあとにします。

## 入力操作
It's not fun to have a game without some sort of movement or action on screen. Let's enable the player's ability to control the bucket. As you know, there are all sorts of ways to get input from a user. We'll focus on a few: the keyboard, mouse, and touch.

We need some way of keeping track of where the player bucket is in the game world. Texture does not store any position state. Sure, you can tell SpriteBatch where to draw it every frame by using the provided overloaded methods. What if you want to rotate it? Resize it? These methods get incredibly complicated the more you want to do.

![long method parameters](/assets/images/dev/a-simple-game/10.png)

Let's use a Sprite instead. Sprite is capable of doing all these things and keeping state. This means that it will remember its properties instead of you having to define them every frame.

```java
public class Main implements ApplicationListener {
    ...
    Sprite bucketSprite; // Declare a new Sprite variable
```

```java
@Override
public void create() {
    ...
    bucketSprite = new Sprite(bucketTexture); // Initialize the sprite based on the texture
    bucketSprite.setSize(1, 1); // Define the size of the sprite
}
```

Erase the `spriteBatch.draw(bucketTexture, 0, 0, 1, 1);` line for the bucket. The Sprite draw code is written in a different way:

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight);
    bucketSprite.draw(spriteBatch); // Sprites have their own draw method

    spriteBatch.end();
}
```
If you run the game again, there shouldn't be any difference in the output. That's good! If you don't see the bucket, make sure you adjusted the line indicated above correctly.

### キーボード操作
Now to capturing player input. This is how you detect if a player is pressing keys on the keyboard. This needs to happen in the input method

```java
private void input() {
    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        // todo: Do something when the user presses the right arrow
    }
}
```

This is known as keyboard polling. Every time a frame is drawn, we're going to check if a key is pressed. There is a list of pretty much every conceivable key in `Gdx.input.Input.Keys`. We want to react to the user pressing the right arrow key.

![keys list](/assets/images/dev/a-simple-game/11.png)

That's great, but what is supposed to happen when the key is pressed? We need to move the coordinates of the bucket sprite.

```java
private void input() {
    float speed = .25f;

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed); // Move the bucket right
    }
}
```

The variable `speed` dictates how fast the bucket moves. Adding to the x makes the bucket move to the right. This basically means "set the bucket x to what it currently is right now plus a little bit more". Subtracting from the x makes the bucket move to the left.

An unfortunate side effect of having our logic inside the render method is that our code behaves differently on different hardware. This is because of differences in framerate. More frames per second means more movement per second.

![fps comparison](/assets/images/dev/a-simple-game/12.png)

To counteract this, we need to use delta time. Delta time is the measured time between frames. If we multiply our movement by delta time, the movement will be consistent no matter what hardware we run this game on.

```java
private void input() {
    float speed = .25f;
    float delta = Gdx.graphics.getDeltaTime(); // retrieve the current delta

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta); // Move the bucket right
    }
}
```

This effectively means that the number we select here is how far the bucket moves in one second. Remember to use delta time whenever you are calculating something that happens over time. Adjust the number to a value that moves the bucket at an acceptable speed like `4f`. Now let's copy the code for movement to the left. Flip the plus to a minus to move to the left.

```java
private void input() {
    float speed = 4f;
    float delta = Gdx.graphics.getDeltaTime();

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta); // move the bucket right
    } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
        bucketSprite.translateX(-speed * delta); // move the bucket left
    }
}
```

Run the game again to make sure you can move the bucket left and right with the keyboard.

### マウス、タッチ操作
Mouse and Touch controls are related. To react to the user clicking or tapping the screen, call the following method:

```java
private void input() {
    float speed = 4f;
    float delta = Gdx.graphics.getDeltaTime();

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta);
    } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
        bucketSprite.translateX(-speed * delta);
    }

    if (Gdx.input.isTouched()) { // If the user has clicked or tapped the screen
        // todo:React to the player touching the screen
    }
}
```

Now the player has clicked the screen, but where did they click? We can use the methods Gdx.input.getX() and Gdx.input.getY() for this. Unfortunately, these values are in window coordinates which don't correlate to our selected pixels per meter. The coordinates are also upside down because Window coordinates start from the top left. We need to declare a Vector2 object to do some math.

```java
public class Main implements ApplicationListener {
    ...
    Vector2 touchPos;
```

Initialize the Vector2:

```java
@Override
public void create() {
    ...

    touchPos = new Vector2();
}
```

Notice that we have created a single instance variable for the Vector2 instead of creating it locally. By reusing this Vector2, we prevent the game from triggering the garbage collector frequently which causes lag spikes in the game. This is how we use the Vector2 to move the bucket:

```java
private void input() {
    float speed = 4f;
    float delta = Gdx.graphics.getDeltaTime();

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta);
    } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
        bucketSprite.translateX(-speed * delta);
    }

    if (Gdx.input.isTouched()) {
        touchPos.set(Gdx.input.getX(), Gdx.input.getY()); // Get where the touch happened on screen
        viewport.unproject(touchPos); // Convert the units to the world units of the viewport
        bucketSprite.setCenterX(touchPos.x); // Change the horizontally centered position of the bucket
    }
}
```

This converts the window coordinates to coordinates in our world space. This code actually supports mobile devices as well, however you should read about [some other input features](/wiki/input/event-handling) that libGDX provides you. Run the game and click the screen to move the bucket.

## ゲームロジック
The player can move left and right now, but they can go completely off the screen. We need to prevent the player from doing that. libGDX provides some helpful methods in the `MathUtils` class. We can achieve our goal by using the `clamp()` method. Remember that the left side of the screen starts at 0. This code detects if the bucket goes too far left. If it does, it snaps its position at the farthest it's allowed to go. Add the following lines to the logic method:

```java
private void logic() {
    // Store the worldWidth and worldHeight as local variables for brevity
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    // Clamp x to values between 0 and worldWidth
    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth));
}
```

Try the game out. This kind of works, but it lets the bucket go just a little too far right. In fact, it's one whole unit too far to the right. This is because the bucket sprite has an origin on the bottom left of the image. To resolve this, we need to subtract the width of the bucket from the right edge.

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    // Store the bucket size for brevity
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    // Subtract the bucket width
    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));
}
```

Now to spawn the rain drops. We will have more than one raindrop, so we need a list to keep track of them. Thankfully, libGDX has many useful [collections](/wiki/utils/collections) to help with this. Declare the list:

```java
public class Main implements ApplicationListener {
    ...
    Array<Sprite> dropSprites;
```

Initialize the list:

```java
public void create() {
    ...

    dropSprites = new Array<>();
}
```

As before, it is advised to organize your code into methods. Create a new method to create a droplet. Place it after your draw method.

```java
private void draw() {
    ...
}

private void createDroplet() {
    // create local variables for convenience
    float dropWidth = 1;
    float dropHeight = 1;
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    
    // create the drop sprite
    Sprite dropSprite = new Sprite(dropTexture);
    dropSprite.setSize(dropWidth, dropHeight);
    dropSprite.setX(0);
    dropSprite.setY(worldHeight);
    dropSprites.add(dropSprite); // Add it to the list
}

@Override
public void pause() {

}
```

The size is the same as the player. Setting the y position at the top of the screen will make it appear as if it's falling from the sky. The `dropSprites.add(dropSprite);` line adds the drop to the list of drops that we can manage in our render loop. 

We'll call `createDroplet()` in the create method.

```java
public void create() {
    ...
    dropSprites = new Array<>();

    createDroplet();
}
```

Drawing each drop is pretty simple. Add the sprite drawing code to the draw method:

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight);
    bucketSprite.draw(spriteBatch);

    // draw each sprite
    for (Sprite dropSprite : dropSprites) {
        dropSprite.draw(spriteBatch);
    }

    spriteBatch.end();
}
```

If you run the program now, you'll see nothing happen. That's because the droplet doesn't have any movement code. Begin coding the logic for the droplets after the bucket logic:

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime(); // retrieve the current delta

    // loop through each drop
    for (Sprite dropSprite : dropSprites) {
        dropSprite.translateY(-2f * delta); // move the drop downward every frame
    }
}
```

We have a problem here. The rain drop only spawns on the left side every time. You could change the position, of course, but there is no variability. No randomness. We need it to be a random position between 0 and the width of the world.

```java
private void createDroplet() {
    float dropWidth = 1;
    float dropHeight = 1;
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    
    Sprite dropSprite = new Sprite(dropTexture);
    dropSprite.setSize(dropWidth, dropHeight);
    dropSprite.setX(MathUtils.random(0f, worldWidth - dropWidth)); // Randomize the drop's x position
    dropSprite.setY(worldHeight);
    dropSprites.add(dropSprite);
}
```

Again, we're subtracting the width of the sprite so none of the raindrops appear outside of the view. Success!

That's only one droplet though. When it rains, we should have multiple droplets over the course of time. Let's move the droplet spawning code to the logic method. This will make droplets repeatedly every frame. Cut the line from the create method:

![Cut the droplet](/assets/images/dev/a-simple-game/13.png)

Paste it into the logic method:

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();

    for (Sprite dropSprite : dropSprites) {
        dropSprite.translateY(-2f * delta);
    }

    // paste the line here
    createDroplet();
}
```

If you run this, you'll see that we have a catastrophe! There are too many droplets.

![too many droplets](/assets/images/dev/a-simple-game/14.png)

There should be a delay between each spawn. Whenever we need something to be done repeatedly over time with a delay, we can create a timer. Declare a new variable to store the time:

```java
public class Main implements ApplicationListener {
    ...
    float dropTimer;
```

`dropTimer` will keep track of how much time has elapsed between each spawn. Modify the code in the logic method:

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();

    for (Sprite dropSprite : dropSprites) {
        dropSprite.translateY(-2f * delta);
    }

    dropTimer += delta; // Adds the current delta to the timer
    if (dropTimer > 1f) { // Check if it has been more than a second
        dropTimer = 0; // Reset the timer
        createDroplet(); // Create the droplet
    }
}
```

`dropTimer` accumulates the time that passes between every frame. If it's been more than a second, it will update the recorded time and proceed to create the droplet. This works as expected now.

![3 droplets](/assets/images/dev/a-simple-game/15.png)

These droplets will fall off the screen never to be seen again. Java doesn't forget though. These droplets will remain in memory forever. If you [profile](https://visualvm.github.io/) your game you'll see that we have a memory leak.

![memory profile](/assets/images/dev/a-simple-game/16.png)

If the player would leave the game on for a really long time, it will crash. So, we should remove the drop sprite from the list when it falls off screen. We need to make some considerable modifications to the logic for loop. Erase your loop in the logic method:

![erase lines](/assets/images/dev/a-simple-game/17.png)

Replace it with the loop indicated below:

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();

    // Loop through the sprites backwards to prevent out of bounds errors
    for (int i = dropSprites.size - 1; i >= 0; i--) {
        Sprite dropSprite = dropSprites.get(i); // Get the sprite from the list
        float dropWidth = dropSprite.getWidth();
        float dropHeight = dropSprite.getHeight();

        dropSprite.translateY(-2f * delta);

        // if the top of the drop goes below the bottom of the view, remove it
        if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

Removing items in a list while you are iterating through it can cause some unforeseen bugs. That's why we are iterating through the list backwards so you don't skip any indexes. Make sure to learn about other [collections](/wiki/utils/collections#specialized-lists) available like the SnapshotArray and the DelayedRemovalArray for more complex projects.

We have made great progress, however the drops don't interact with the bucket. This is where we incorporate some rudimentary collision detection. This can be achieved with the Rectangle class. We need two rectangles to make comparisons. One for the bucket and one to be reused with every drop.

```java
public class Main implements ApplicationListener {
    ...
    Rectangle bucketRectangle;
    Rectangle dropRectangle;
```

```java
@Override
public void create() {
    ...

    bucketRectangle = new Rectangle();
    dropRectangle = new Rectangle();
}
```

The code now sets the rectangles to the position and dimensions of the Sprites.

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();
    // Apply the bucket position and size to the bucketRectangle
    bucketRectangle.set(bucketSprite.getX(), bucketSprite.getY(), bucketWidth, bucketHeight);

    for (int i = dropSprites.size - 1; i >= 0; i--) {
        Sprite dropSprite = dropSprites.get(i);
        float dropWidth = dropSprite.getWidth();
        float dropHeight = dropSprite.getHeight();

        dropSprite.translateY(-2f * delta);
        // Apply the drop position and size to the dropRectangle
        dropRectangle.set(dropSprite.getX(), dropSprite.getY(), dropWidth, dropHeight);

        if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
        else if (bucketRectangle.overlaps(dropRectangle)) { // Check if the bucket overlaps the drop
            dropSprites.removeIndex(i); // Remove the drop
        }
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

`bucketRectangle.overlaps(dropRectangle)` checks if the bucket overlaps the drop. If it does, the Sprite will be removed from the list of drop Sprites. That means it will no longer be drawn or acted upon. It simply doesn't exist anymore, making it look like the bucket collected it. Make sure your game works as expected. You're almost done!

## 効果音と音楽
It's very easy to add a line to play a sound effect now that we are at the end of our workflow. We want the drop sound (the sound effect loaded at the beginning of this tutorial) to play when the bucket collides with the drop. It should not play when the drop falls out of the level.

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();
    bucketRectangle.set(bucketSprite.getX(), bucketSprite.getY(), bucketWidth, bucketHeight);

    for (int i = dropSprites.size - 1; i >= 0; i--) {
        Sprite dropSprite = dropSprites.get(i);
        float dropWidth = dropSprite.getWidth();
        float dropHeight = dropSprite.getHeight();

        dropSprite.translateY(-2f * delta);
        dropRectangle.set(dropSprite.getX(), dropSprite.getY(), dropWidth, dropHeight);
        
        if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
        else if (bucketRectangle.overlaps(dropRectangle)) {
            dropSprites.removeIndex(i);
            dropSound.play(); // Play the sound
        }
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

The music should play at the beginning of the game. It needs to loop continuously until the player is done playing the game. The file is also a little loud, so it should play at half volume. Volume is a value from 0f to 1f with 1f being the normal volume of the file.

```java
@Override
public void create() {
    ...

    music.setLooping(true);
    music.setVolume(.5f);
    music.play();
}
```

Make sure your speakers are on and try it out.

## さらに学ぶには
So, you're at the final steps of making a game. You should test the game out. Tweak values to make the game easier or harder. This can be done by changing how fast the bucket moves and what rate the droplets spawn.

If you want to let your friends and colleagues try your game out, you'll need to make a distributable that they can play. No one is going to want set up an IDE and copy your entire project just to play it. See the page on [Deploying your application](/wiki/deployment/deploying-your-application).

Now that you've completed the simple game, it's time to [extend the simple game](/wiki/start/simple-game-extended). This project managed to put all of its code in a single class. This was in the service of making it simple, but it is a terrible way to organize code. The next tutorial will teach you about the Game class and how to implement Screen to arrange your project. It will also cover other important improvements to your game. For example, these instructions skipped the use of the dispose() method because it's not relevant for single page project. When working with multiple screens, you may want to dispose of resources from the last screen to release the memory for new resources in your game.

Game design is a constant journey of learning. The [wiki](/wiki/) goes further in depth regarding all the subjects you have learned here. Look into [collections](/wiki/utils/collections), [TexturePacker](/wiki/tools/texture-packer), [AssetManager](/wiki/managing-your-assets), [audio](/wiki/audio/audio), [memory management](/wiki/articles/memory-management), and [user input](/wiki/input/input-handling).

This tutorial focused entirely on desktop development. There are many more considerations you must make before you explore Android, iOS, and HTML5 development. There is an extensive article on [considerations for HTML5](/wiki/html5-backend-and-gwt-specifics), for example. Java is not truly "write once, run anywhere" but libGDX takes you pretty close to that goal.

## 完全なサンプルコード
The following is the full example code of the game described throughout this tutorial. It's here for reference if you get stuck on any of the steps. Don't cheat yourself by copying the whole thing! Learning how to program is mainly you asking yourself questions and trying to resolve issues on your own first.

```java
package com.badlogic.drop;

import com.badlogic.gdx.ApplicationListener;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.audio.Music;
import com.badlogic.gdx.audio.Sound;
import com.badlogic.gdx.graphics.Color;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.Sprite;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.MathUtils;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Vector2;
import com.badlogic.gdx.utils.Array;
import com.badlogic.gdx.utils.ScreenUtils;
import com.badlogic.gdx.utils.viewport.FitViewport;

public class Main implements ApplicationListener {
    Texture backgroundTexture;
    Texture bucketTexture;
    Texture dropTexture;
    Sound dropSound;
    Music music;
    SpriteBatch spriteBatch;
    FitViewport viewport;
    Sprite bucketSprite;
    Vector2 touchPos;
    Array<Sprite> dropSprites;
    float dropTimer;
    Rectangle bucketRectangle;
    Rectangle dropRectangle;

    @Override
    public void create() {
        backgroundTexture = new Texture("background.png");
        bucketTexture = new Texture("bucket.png");
        dropTexture = new Texture("drop.png");
        dropSound = Gdx.audio.newSound(Gdx.files.internal("drop.mp3"));
        music = Gdx.audio.newMusic(Gdx.files.internal("music.mp3"));
        spriteBatch = new SpriteBatch();
        viewport = new FitViewport(8, 5);
        bucketSprite = new Sprite(bucketTexture);
        bucketSprite.setSize(1, 1);
        touchPos = new Vector2();
        dropSprites = new Array<>();
        bucketRectangle = new Rectangle();
        dropRectangle = new Rectangle();
        music.setLooping(true);
        music.setVolume(.5f);
        music.play();
    }

    @Override
    public void resize(int width, int height) {
        viewport.update(width, height, true);
    }

    @Override
    public void render() {
        input();
        logic();
        draw();
    }

    private void input() {
        float speed = 4f;
        float delta = Gdx.graphics.getDeltaTime();

        if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
            bucketSprite.translateX(speed * delta);
        } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
            bucketSprite.translateX(-speed * delta);
        }

        if (Gdx.input.isTouched()) {
            touchPos.set(Gdx.input.getX(), Gdx.input.getY());
            viewport.unproject(touchPos);
            bucketSprite.setCenterX(touchPos.x);
        }
    }

    private void logic() {
        float worldWidth = viewport.getWorldWidth();
        float worldHeight = viewport.getWorldHeight();
        float bucketWidth = bucketSprite.getWidth();
        float bucketHeight = bucketSprite.getHeight();

        bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

        float delta = Gdx.graphics.getDeltaTime();
        bucketRectangle.set(bucketSprite.getX(), bucketSprite.getY(), bucketWidth, bucketHeight);

        for (int i = dropSprites.size - 1; i >= 0; i--) {
            Sprite dropSprite = dropSprites.get(i);
            float dropWidth = dropSprite.getWidth();
            float dropHeight = dropSprite.getHeight();

            dropSprite.translateY(-2f * delta);
            dropRectangle.set(dropSprite.getX(), dropSprite.getY(), dropWidth, dropHeight);

            if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
            else if (bucketRectangle.overlaps(dropRectangle)) {
                dropSprites.removeIndex(i);
                dropSound.play();
            }
        }

        dropTimer += delta;
        if (dropTimer > 1f) {
            dropTimer = 0;
            createDroplet();
        }
    }

    private void draw() {
        ScreenUtils.clear(Color.BLACK);
        viewport.apply();
        spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
        spriteBatch.begin();

        float worldWidth = viewport.getWorldWidth();
        float worldHeight = viewport.getWorldHeight();

        spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight);
        bucketSprite.draw(spriteBatch);

        for (Sprite dropSprite : dropSprites) {
            dropSprite.draw(spriteBatch);
        }

        spriteBatch.end();
    }

    private void createDroplet() {
        float dropWidth = 1;
        float dropHeight = 1;
        float worldWidth = viewport.getWorldWidth();
        float worldHeight = viewport.getWorldHeight();

        Sprite dropSprite = new Sprite(dropTexture);
        dropSprite.setSize(dropWidth, dropHeight);
        dropSprite.setX(MathUtils.random(0f, worldWidth - dropWidth));
        dropSprite.setY(worldHeight);
        dropSprites.add(dropSprite);
    }

    @Override
    public void pause() {
        
    }

    @Override
    public void resume() {
        
    }

    @Override
    public void dispose() {
        
    }
}
```
