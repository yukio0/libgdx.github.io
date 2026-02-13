---
title: Gdx freetype
---
## はじめに

ゲーム内で文字を描画したい場合、通常は[BitmapFont](/wiki/graphics/2d/fonts/bitmap-fonts)を使います。
しかし、欠点があります。

* **BitmapFontは画像に依存するため、別サイズで表示したいときに拡大縮小が必要になり、見た目が崩れることがあります。**

では、ゲームで必要な最大サイズのBitmapFontだけを用意しておき、拡大はせず縮小だけにすればいいのでは？
ええ、確かにそのとおりです。ただし大きく縮小すると、ミップマップの有無によって「ジャギーが目立つ」か「少しぼやける」かのどちらかになりがちです。
この問題を解決しようとするのが[距離場フォント](https://libgdx.com/wiki/graphics/2d/fonts/distance-field-fonts)（Distance field fonts）ですが、このページの主題はそれではありません！

また、BitmapFontは対応するTrueTypeフォント（.ttf）より保存容量を多く消費することがあります。ただし、それらのフォントがgdx-freetype自体より容量を食うかどうかは、ゲーム内容やターゲットプラットフォーム次第です。

そこで解決策となるのが`gdx-freetype`拡張です。
  * 軽量な .ttf ファイルだけをゲームに同梱する
  * 必要なサイズのBitmapFontを実行時に生成する
  * ユーザーが自分のフォントをゲームに追加できるようにする

チュートリアル： [https://libgdxinfo.wordpress.com](https://libgdxinfo.wordpress.com/basic-label/)

## 詳細

これは拡張機能なので、libGDXプロジェクトにデフォルトでは含まれていません。追加方法は、プロジェクトの構成によって異なります。

### プロジェクトにgdx-freetypeを追加する方法

#### Gradleを使うプロジェクト

新規プロジェクトの場合は、[セットアップツール](https://libgdx.com/dev/project-generation/)のextensionsからFreetypeを選ぶだけでOKです。

既存のGradleプロジェクトに追加する場合は、[Gradleによる依存関係管理](/wiki/articles/dependency-management-with-gradle#freetypefont-gradle)を参照してください。

#### HTML5

gdx-freetype はHTML5に対応していません。ただし、Intrigusによる[gdx-freetype-gwt](https://github.com/intrigus/gdx-freetype-gwt)を使えば、HTML5でも利用可能になります。バージョン1.9.10.1は、libGDX 1.10.0を含む新しいバージョンとも互換性があります。

### コードでgdx-freetypeを使う方法

gdx-freetype拡張をコードで使うのはとても簡単です。

```java
FreeTypeFontGenerator generator = new FreeTypeFontGenerator(Gdx.files.internal("fonts/myfont.ttf"));
FreeTypeFontParameter parameter = new FreeTypeFontParameter();
parameter.size = 12;
BitmapFont font12 = generator.generateFont(parameter); // フォントサイズ 12 ピクセル
generator.dispose(); // メモリリークを避けるため、disposeを忘れずに！
```
もっと手軽に表示したいなら、フォントファイルをプロジェクトのassetsフォルダに置いてしまう方法があります。その場合、上のコードの1行目を修正し、パラメータにフォントファイル名だけを指定すればOKです。

[FreeTypeFontParameter](https://github.com/libgdx/libgdx/blob/master/extensions/gdx-freetype/src/com/badlogic/gdx/graphics/g2d/freetype/FreeTypeFontGenerator.java)のデフォルト値は次のとおりです。
```java
/** サイズ（ピクセル単位） */
public int size = 16;
/** 前景色（黒以外の縁取りを使う場合は必須） */
public Color color = Color.WHITE;
/** 縁取りの太さ（ピクセル単位）。0で無効 */
public float borderWidth = 0;
/** 縁取りの色。borderWidth > 0 のときのみ使用 */
public Color borderColor = Color.BLACK;
/** trueで角ばった（マイター結合の）縁取り、falseで丸い縁取り */
public boolean borderStraight = false;
/** 文字影のX方向オフセット（ピクセル単位）。0で無効 */
public int shadowOffsetX = 0;
/** 文字影のY方向オフセット（ピクセル単位）。0で無効 */
public int shadowOffsetY = 0;
/** 影の色。shadowOffset > 0 のときのみ使用 */
public Color shadowColor = new Color(0, 0, 0, 0.75f);
/** フォントに含める文字セット */
public String characters = DEFAULT_CHARS;
/** カーニング（字間調整）を含めるかどうか */
public boolean kerning = true;
/** 使用するPixmapPacker（任意） */
public PixmapPacker packer = null;
/** フォントを縦方向に反転するかどうか */
public boolean flip = false;
/** 生成されるテクスチャにミップマップを生成するかどうか */
public boolean genMipMaps = false;
/** 縮小時のフィルタ */
public TextureFilter minFilter = TextureFilter.Nearest;
/** 拡大時のフィルタ */
public TextureFilter magFilter = TextureFilter.Nearest;
```

大きなフォントを描画する場合、デフォルトのPixmapPackerのページサイズでは小さすぎることがあります。その場合は自分でPixmapPackerを渡すか、`FreeTypeFontGenerator.setMaxTextureSize`を使ってデフォルトのページサイズを変更してください。

ウィンドウ解像度の違いを拡大縮小で吸収したくない場合は、ゲームの`resize()`イベントでフォントを生成するのも手です（ただし古いBitmapFontを必ずdisposeしてください）。特にgdx-freetype-gwtでは、ページサイズが扱える範囲にフォントサイズを収めないと、文字化けやクラッシュの原因になります。

### 例

```java
parameter.borderColor = Color.BLACK;
parameter.borderWidth = 3;
```
![images/border.png](/assets/wiki/images/border.png)

```java
parameter.shadowColor = Color.BLACK;
parameter.shadowOffsetX = 3;
parameter.shadowOffsetY = 3;
```
![images/shadow.png](/assets/wiki/images/shadow.png)

FreeType拡張で生成した`BitmapFont`は`AssetManager`から読み込むこともできます。詳しくは[FreeTypeFontLoaderTest](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/extensions/FreeTypeFontLoaderTest.java)を参照してください。

### 注意点

以下は、[https://web.archive.org/web/20201128081723/https://www.badlogicgames.com/wordpress/?p=2300](https://web.archive.org/web/20201128081723/https://www.badlogicgames.com/wordpress/?p=2300)からの引用です。
  * アジア圏の文字は「動くかもしれない」が、上記の注意点のとおり。グリフ数が多すぎます。解決策を考えています。
  * アラビア語のような右から左へ書く文字は不可です。BitmapFontとBitmapFontCacheのレイアウト「アルゴリズム」では扱えません。
  * FreeTypeにどんなフォントでも投げればOK、というわけではありません。世の中にはひどいフォントもあり、ヒンティング情報が不十分だったり欠けていたりして、見た目が残念なことになります。

----

[サンプル](https://hg.sr.ht/~dermetfan/somelibgdxtests)をダウンロードできます。
