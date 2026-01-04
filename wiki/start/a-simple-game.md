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

OpenGLベースのゲーム（libGDX で作られたものなど）では、小数は通常`22.5f`のような浮動小数点リテラルで表現されます。「f」を付け忘れるとビルドエラーになることがあります。`float`が`double`（例：`22.5`）より推奨されるのは、メモリ使用量が少なく、多くのハードウェアでサポートされ、OpenGLが通常`float`を期待するためです。IDEAでは、`ctrl+p`を押すことで、メソッドが期待する引数を確認できます。

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

管理すべきアセットが多くなってきました。そういうときは[アセット管理](/wiki/managing-your-assets)を使うべきです。アセット管理もwikiに記載されています。ただし、繰り返しになりますが、これは本チュートリアルの範囲外です。シンプルなままにしましょう。

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

また、beginとendは必ず正しい順序で呼んでください。スプライトバッチをbeginとendの外で描画してはいけません。そうした場合はエラーメッセージが表示されます。

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
画面上で何の動きやアクションもないゲームは楽しくないですね。プレイヤーがバケツを操作できるようにしましょう。ご存じの通り、ユーザーから入力を受け取る方法にはさまざまなものがあります。ここでは、キーボード、マウス、タッチ操作に焦点を当てます。

ゲーム世界の中でプレイヤーのバケツがどこにあるのかを管理する仕組みが必要です。テクスチャは位置情報を保持しません。確かに、SpriteBatchクラスに用意されているオーバーロードされたメソッドを使えば、毎フレームどこに描画するかを指定することはできます。しかし、もし回転させたい場合はどうでしょう？サイズを変えたくなったら？こうしたことをやろうとすると、描画用のメソッドはやりたいことが増えるほど急激に複雑になっていきます。

![long method parameters](/assets/images/dev/a-simple-game/10.png)

スプライトを使いましょう。スプライトはこれらすべての操作を行うことができ、しかも状態を保持してくれます。つまり、毎フレームこちらが設定し直さなくても、位置、サイズ、回転といったプロパティを自分で覚えていてくれるのです。

```java
public class Main implements ApplicationListener {
    ...
    Sprite bucketSprite; // 新しいSprite変数を宣言
```

```java
@Override
public void create() {
    ...
    bucketSprite = new Sprite(bucketTexture); // テクスチャを元にスプライトを初期化します
    bucketSprite.setSize(1, 1); // スプライトのサイズを定義します
}
```

バケツを描画していた`spriteBatch.draw(bucketTexture, 0, 0, 1, 1);`の行を消してください。スプライトを使う場合、描画方法は少し変わります。

```java
private void draw() {
    ScreenUtils.clear(Color.BLACK);
    viewport.apply();
    spriteBatch.setProjectionMatrix(viewport.getCamera().combined);
    spriteBatch.begin();

    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    spriteBatch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight);
    bucketSprite.draw(spriteBatch); // スプライトは専用のdrawメソッドを持っています

    spriteBatch.end();
}
```
この状態で、もう一度ゲームを実行してみてください。表示結果は見た目上は何も変わらないはずです。それで正解です！もしバケツが表示されなかった場合、バケツを描画している行を正しく修正できているか確認してみてください。

### キーボード操作
では、プレイヤーの入力を取得していきましょう。キーボード入力を検知する方法です。これはinputメソッドの中で行う必要があります。

```java
private void input() {
    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        // TODO: ユーザーが右矢印キーを押したときの処理
    }
}
```

この方法はキーボードポーリングと呼ばれています。毎フレーム描画されるたびにキーが押されているかどうかを確認するというやり方です。`Gdx.input.Input.Keys`には、考えうるほぼすべてのキーが定義されています。今回、ユーザーが右矢印キーを押したことに反応したいわけです。

![keys list](/assets/images/dev/a-simple-game/11.png)

ここまでは良いですね。でも、キーが押されたら、何を起こすべきでしょうか？バケツのSpriteの座標を動かす必要がありますね。

```java
private void input() {
    float speed = .25f;

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed); // バケツを右に動かします
    }
}
```

この`speed`という変数はバケツがどれくらいの速さで動くかを決めています。x座標に値を足すと右へ移動します。つまりこのコードは、「バケツのx座標に、今の値から少し足した値を設定する」と言っているのと基本的に同じ意味です。x座標に値を引くとバケツは左に移動します。

ロジックをrenderメソッドの中に書いていると、残念な副作用として、ハードウェアごとにコードの挙動が変わってしまいます。これはフレームレートの違いが原因です。1秒あたりのフレーム数が多いほど、1秒間に行われる移動量も多くなってしまいます。

![fps comparison](/assets/images/dev/a-simple-game/12.png)

これを防ぐために、デルタタイムを使用する必要があります。デルタタイムとは、前のフレームから次のフレームまでにかかった時間を表します。移動量にデルタタイムを掛けることで、どのハードウェアでゲームを実行しても、動きが一定になります。

```java
private void input() {
    float speed = .25f;
    float delta = Gdx.graphics.getDeltaTime(); // 現在のデルタタイムを取得します

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta); // バケツを右に動かします
    }
}
```

実質、ここで指定した数値`speed`は「1秒間にバケツがどれだけ移動するか」を表しています。時間の経過に伴って起こる処理を計算するときは、デルタタイムを使うことを忘れないでください。数値は`4f`のようなバケツがちょうどよい速さで動く値に調整しましょう。右移動のコードをコピーして、左へ移動する処理も追加します。プラスをマイナスに変えれば、左方向に動くようになります。

```java
private void input() {
    float speed = 4f;
    float delta = Gdx.graphics.getDeltaTime();

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta); // バケツを右に動かします
    } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
        bucketSprite.translateX(-speed * delta); // バケツを左に動かします
    }
}
```

もう一度ゲームを実行して、キーボード操作でバケツを左右に動かせるか確認してください。

### マウス、タッチ操作
マウス操作とタッチ操作は密接に関係しています。ユーザーが画面をクリック、またはタップしたことに反応するためには、次のメソッドを呼び出します。

```java
private void input() {
    float speed = 4f;
    float delta = Gdx.graphics.getDeltaTime();

    if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
        bucketSprite.translateX(speed * delta);
    } else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
        bucketSprite.translateX(-speed * delta);
    }

    if (Gdx.input.isTouched()) { // ユーザーが画面をクリック、またはタップした場合
        // TODO: ユーザーが画面をクリック、またはタップしたときの処理を書く
    }
}
```

さて、プレイヤーが画面をクリックしたことは分かりました。しかし「どこをクリックしたのか？」はまだ分かりません。これを知るために、Gdx.input.getX()メソッドとGdx.input.getY()メソッドを使います。しかしながら、これらの値はウィンドウ座標で取得されるため、私たちが決めた「1メートルあたりのピクセル数」とは対応していません。さらに、ウィンドウ座標は左上が原点になっているため、ゲーム世界の座標系とは上下が逆になっています。計算用としてVector2オブジェクトを宣言する必要があります。

```java
public class Main implements ApplicationListener {
    ...
    Vector2 touchPos;
```

次にVector2を初期化します。

```java
@Override
public void create() {
    ...

    touchPos = new Vector2();
}
```

ここで注目してほしいのは、Vector2をローカル変数として毎回生成するのではなく、インスタンス変数としてひとつだけ作成している点です。Vector2を再利用することで、ゲームが頻繁にガベージコレクタ（GC）を動かし、それによってゲームラグが起きることを防ぎます。次のコードが、Vector2を使ってバケツを移動させる方法です。

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
        touchPos.set(Gdx.input.getX(), Gdx.input.getY()); // 画面上でタッチ（クリック）された位置を取得します
        viewport.unproject(touchPos); // ウィンドウ座標をゲーム世界の座標に変換します
        bucketSprite.setCenterX(touchPos.x); // バケツの中心をタッチ（クリック）された位置のX座標に来るように変更します
    }
}
```

これはウィンドウ座標をゲーム世界の座標に変換する処理です。このコードはモバイル端末にも対応していますが、libGDXが提供する[そのほかの入力機能](/wiki/input/event-handling)も確認してください。ゲームを実行し、画面をクリック（またはタップ）してバケツが動くか試してみましょう。

## ゲームロジック
プレイヤーは左右に動かすことができるようになりましたが、今のままだと画面の外まで移動できてしまいます。それは防ぐ必要があります。libGDXは役立つメソッドがある`MathUtils`クラスを提供しています。ここでは`clamp()`メソッドを使って目的を達成します。画面の左端は0から始まることを思い出してください。このコードはバケツが左に行きすぎていないかをチェックします。もし行きすぎていた場合、移動できる最も左の位置に留めます。次の行をlogicメソッドに追加してください。

```java
private void logic() {
    // 可読性のため、worldWidthとworldHeightをローカル変数にします
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    // x座標を0からworldWidthの範囲に収めます
    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth));
}
```

実際にゲームを動かしてみてください。ある程度はうまく動きますが、右方向に関してはバケツが少しだけ行きすぎてしまいます。これはバケツのスプライトの原点がバケツ画像の左下にあるためです。この問題を解決するには、バケツの幅を右端の制限から差し引く必要があります。

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();

    // 可読性のため、バケツのサイズをローカル変数にします
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    // バケツの幅を差し引きます
    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));
}
```

次は、雨粒を生成していきましょう。雨粒は1つだけではなく、複数存在するため、管理するためのリストが必要になります。幸いなことに、libGDXにはこの目的に使える便利な[コレクション](/wiki/utils/collections)が数多く用意されています。まずは、そのリストを宣言しましょう。

```java
public class Main implements ApplicationListener {
    ...
    Array<Sprite> dropSprites;
```

次にリストを初期化します。

```java
public void create() {
    ...

    dropSprites = new Array<>();
}
```

これまでと同様に、コードはメソッドごとに整理することをおすすめします。雨粒を生成するための新しいメソッドを作成しましょう。drawメソッドの後に配置してください。

```java
private void draw() {
    ...
}

private void createDroplet() {
    // 使いやすくするためのローカル変数を作成します
    float dropWidth = 1;
    float dropHeight = 1;
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    
    // 雨粒のスプライトを作成します
    Sprite dropSprite = new Sprite(dropTexture);
    dropSprite.setSize(dropWidth, dropHeight);
    dropSprite.setX(0);
    dropSprite.setY(worldHeight);
    dropSprites.add(dropSprite); // リストに追加します
}

@Override
public void pause() {

}
```

サイズはバケツと同じです。y座標を画面の一番上に設定することで、雨粒が空から落ちてくるように見せることができます。`dropSprites.add(dropSprite);`の行では、雨粒を「renderループの中で制御できる雨粒リスト」に追加しています。

次に、create()メソッドの中で`createDroplet()`を呼び出しましょう。

```java
public void create() {
    ...
    dropSprites = new Array<>();

    createDroplet();
}
```

雨粒を1つずつ描画するのはとても簡単です。次のように、スプライトの描画処理をdrawメソッドに追加してください。

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

    // 各雨粒のスプライトを描画します
    for (Sprite dropSprite : dropSprites) {
        dropSprite.draw(spriteBatch);
    }

    spriteBatch.end();
}
```

この時点でプログラムを実行しても、何も起こらないように見えるはずです。雨粒にまだ動きの処理が書かれていないためです。バケツのロジックの後に、雨粒のロジックを書き始めましょう。

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime(); // 現在のデルタタイムを取得します

    // 各雨粒についてループ処理を行います
    for (Sprite dropSprite : dropSprites) {
        dropSprite.translateY(-2f * delta); // 毎フレーム、雨粒を下方向に移動させます
    }
}
```

ここで問題があります。雨粒は毎回画面の左側だけにしか生成されていません。位置を変更することはできますが、それでは変化がなく、ランダム性がありません。必要なのはゲーム世界の幅の中でランダムな位置に雨粒を生成することです。

```java
private void createDroplet() {
    float dropWidth = 1;
    float dropHeight = 1;
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    
    Sprite dropSprite = new Sprite(dropTexture);
    dropSprite.setSize(dropWidth, dropHeight);
    dropSprite.setX(MathUtils.random(0f, worldWidth - dropWidth)); // 雨粒のX座標をランダムに設定します
    dropSprite.setY(worldHeight);
    dropSprites.add(dropSprite);
}
```

バケツのときと同様に、スプライトの幅を差し引いています。こうすることで、雨粒が画面の外に生成されることがなくなります。成功です！

しかしながら、雨粒は1つだけです。雨が降るとき、時間の経過とともに複数の雨粒が降ってくるべきです。そこで、雨粒を生成するコードをlogicメソッドに移動しましょう。こうすることで、毎フレーム雨粒が生成されるようになります。createメソッドから次の行を切り取ってください。

![Cut the droplet](/assets/images/dev/a-simple-game/13.png)

それをlogicメソッドの中に貼り付けます。

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

    // ここに貼り付けます
    createDroplet();
}
```

これを実行すると、大惨事が起きていることが分かるはずです！雨粒が多すぎます。

![too many droplets](/assets/images/dev/a-simple-game/14.png)

雨粒が生成されるたびに少し間隔があるべきです。このように「一定間隔で、繰り返し何かを行いたい」場合、タイマーを使うのが定石です。経過時間を保持するための新しい変数を宣言しましょう。

```java
public class Main implements ApplicationListener {
    ...
    float dropTimer;
```

`dropTimer`は雨粒が生成されてから次に生成されるまでに、どれだけ時間が経過したかを追跡するための変数です。次に、logicメソッド内のコードを次のように修正します。

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

    dropTimer += delta; // 現在のデルタタイムをタイマーに加算します
    if (dropTimer > 1f) { // 1秒以上経過したかを確認します
        dropTimer = 0; // タイマーをリセットします
        createDroplet(); // 新しい雨粒を生成します
    }
}
```

`dropTimer`はフレームごとの経過時間を蓄積していきます。その合計が1秒を超えた場合、記録されている時間を更新し、新しい雨粒の生成へと進みます。これで期待どおりの挙動になりました。

![3 droplets](/assets/images/dev/a-simple-game/15.png)

これらの雨粒は画面の外へ落ちていき、二度と表示されることはありません。しかしながら、Javaは忘れてくれません。画面から消えた雨粒たちは、メモリ上にはずっと残り続けてしまいます。ゲームを[プロファイル](https://visualvm.github.io/)で解析してみると、メモリリークが発生していることが確認できるはずです。

![memory profile](/assets/images/dev/a-simple-game/16.png)

プレイヤーがゲームを非常に長い時間起動したままにしていたら、最終的にはクラッシュしてしまいます。そのため、画面の外に落ちた雨粒のスプライトは、リストから削除する必要があります。これを行うにはlogicメソッド内のforループの処理を大きく書き換える必要があります。logicメソッドの中にある現在のループを削除してください。

![erase lines](/assets/images/dev/a-simple-game/17.png)

次のループに置き換えてください。

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();

    // 配列の範囲外エラーを防ぐため、後ろから順にスプライトを処理します
    for (int i = dropSprites.size - 1; i >= 0; i--) {
        Sprite dropSprite = dropSprites.get(i); // リストからスプライトを取得します
        float dropWidth = dropSprite.getWidth();
        float dropHeight = dropSprite.getHeight();

        dropSprite.translateY(-2f * delta);

        // 雨粒の上端が画面の下端より下に行ったら削除します
        if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

リストを走査している最中に要素を削除すると、予期しないバグを引き起こすことがあります。そのため、リストを後ろから順にループすることで、インデックスを飛ばしてしまわないようにしています。より複雑なプロジェクトでは、SnapshotArrayやDelayedRemovalArrayなど、ほかの[コレクション](/wiki/utils/collections#specialized-lists)についてもぜひ学んでおいてください。

ここまでで大きな前進を遂げましたが、まだ雨粒はバケツと相互作用していません。そこで基本的な衝突検出を組み込みます。これは長方形（Rectangle）クラスを使うことで実現できます。比較のために、2つの長方形が必要になります。1つはバケツ用、そしてもう1つは、すべての雨粒に対して使い回すためのものです。

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

このコードでは、スプライトの位置と大きさをそのまま長方形に反映させています。

```java
private void logic() {
    float worldWidth = viewport.getWorldWidth();
    float worldHeight = viewport.getWorldHeight();
    float bucketWidth = bucketSprite.getWidth();
    float bucketHeight = bucketSprite.getHeight();

    bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));

    float delta = Gdx.graphics.getDeltaTime();
    // バケツの位置とサイズをbucketRectangle（バケツ用長方形）に反映させます。
    bucketRectangle.set(bucketSprite.getX(), bucketSprite.getY(), bucketWidth, bucketHeight);

    for (int i = dropSprites.size - 1; i >= 0; i--) {
        Sprite dropSprite = dropSprites.get(i);
        float dropWidth = dropSprite.getWidth();
        float dropHeight = dropSprite.getHeight();

        dropSprite.translateY(-2f * delta);
        // 雨粒の位置とサイズをdropRectangle（雨粒用長方形）に反映させます
        dropRectangle.set(dropSprite.getX(), dropSprite.getY(), dropWidth, dropHeight);

        if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
        else if (bucketRectangle.overlaps(dropRectangle)) { // バケツと雨粒が重なっているかをチェックします
            dropSprites.removeIndex(i); // 雨粒を削除します
        }
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

`bucketRectangle.overlaps(dropRectangle)`はバケツと雨粒が重なっているかどうかを判定します。もし重なっていれば、その雨粒のスプライトは雨粒リストから削除されます。リストから削除されたスプライトはもう描画されることもなく、ロジックの処理対象にもなりません。完全に存在しなくなり、バケツが雨粒をキャッチしたように見えます。ゲームが意図した通りに動いているかを確認してください。ゲーム作成の完了はあと一歩です！

## 効果音と音楽
一連の作業の終盤まで来た今、効果音を再生する行を追加するのはとても簡単です。バケツが雨粒と衝突したときに、チュートリアルの冒頭で読み込んだ雨粒の効果音を再生したいと思います。ただし、雨粒が画面の外に落ちただけの場合には再生しないようにする必要があります。

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
            dropSound.play(); // 効果音を鳴らします
        }
    }

    dropTimer += delta;
    if (dropTimer > 1f) {
        dropTimer = 0;
        createDroplet();
    }
}
```

ゲーム開始と同時に音楽を再生しましょう。この音楽はプレイヤーがゲームを遊んでいる間、ずっとループ再生される必要があります。また、この音楽ファイルは少し音量が大きいので、音量は半分（0.5f）に設定します。音量は0fから1fの範囲で指定し、1fが元の音量です。

```java
@Override
public void create() {
    ...

    music.setLooping(true);
    music.setVolume(.5f);
    music.play();
}
```

スピーカーがオンになっていることを確認して、実際に動かしてみてください。

## さらに学ぶには
さあ、いよいよゲーム作成の最終段階です。実際にゲームを動かしてテストしてみましょう。ゲームの難易度は数値調整で大きく変わります。例えば、バケツの移動速度と雨粒の生成間隔を変更することでゲーム難易度を変更できます。

友人や同僚にあなたのゲームを遊んでもらいたいなら、プレイ可能な配布用の形式でゲームを作成する必要があります。ゲームを遊ぶために、IDEを準備してプロジェクト全体をコピーするような手間をかける人は誰もいないでしょう。[アプリケーションのデプロイ](/wiki/deployment/deploying-your-application)のページを参考にしてください。

シンプルなゲームの作成が完了したので、次は[シンプルなゲームの拡張](/wiki/start/simple-game-extended)に挑戦してみましょう。このプロジェクトはすべてのコードを単一のクラスに収めました。これは簡単にするためでしたが、コードを整理する方法としては最悪の方法です。次のチュートリアルでは、Gameクラスとプロジェクトを構成するためのScreenの実装方法について説明します。また、ゲームにおけるその他の重要な改善点についても取り上げます。例えば、これらの手順では dispose() メソッドの使用を省略しました。これは1つの画面しかないプロジェクトには関係がないためです。複数の画面を扱う場合、新しいリソースのためにメモリを解放するため、前の画面のリソースを破棄したい場合があります。

ゲームデザインは絶え間ない学びの旅です。今回ここで学んだ内容について、[wiki](/wiki/)ではさらに深く掘り下げた解説が用意されています。[コレクション](/wiki/utils/collections)、[TexturePacker](/wiki/tools/texture-packer)、[アセット管理](/wiki/managing-your-assets)、[オーディオ](/wiki/audio/audio)、[メモリ管理](/wiki/articles/memory-management)、[入力処理](/wiki/input/input-handling)をご覧してください。

このチュートリアルはデスクトップ向け開発に完全に焦点を当てていました。しかし、Android、iOS、HTML5といった他のプラットフォームに進む前には、さらに多くの点を考慮する必要があります。例えば、[HTML5開発における考慮事項](/wiki/html5-backend-and-gwt-specifics)について詳細な記事があります。Javaは真の意味では「Write once, run anywhere」な言語ではありませんが、libGDXはその理想にかなり近づけてくれる存在です。

## 完全なサンプルコード
以下はこのチュートリアル全体を通して作ってきたゲームの完全なサンプルコードです。どこかの手順で詰まってしまったときの参照用として置いてあります。全部丸写しするようなことはやめましょう！プログラミングを学ぶとは、自分自身に問いを投げて、まず自分で問題を解決しようと試みることです。

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
