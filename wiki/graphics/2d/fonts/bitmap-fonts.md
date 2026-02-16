---
title: ビットマップフォント
---
libGDXは、フォントの描画にビットマップ画像ファイル（PNG）を利用します。フォント内の各グリフ（文字）には、対応する`TextureRegion`が割り当てられています。

[BitmapFontクラス](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/graphics/g2d/BitmapFont.html) [(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/graphics/g2d/BitmapFont.java)

`BitmapFont`はlibGDX 1.5.6のリリースでリファクタリングされました。変更点の詳細と、1.5.6未満のコードから新APIへ移行する小さな例については、[このブログ記事](https://web.archive.org/web/20200928220256/https://www.badlogicgames.com/wordpress/?p=3658)を参照してください。

`BitmapFont`の使い方に関するチュートリアルは、[https://libgdxinfo.wordpress.com](https://libgdxinfo.wordpress.com/basic-label/)にあります。

## フォントファイルのファイル形式仕様

参照によると、BMFontはもともと[AngelCode](https://www.angelcode.com/)のAndreas Jönsson によって作られたとされています。

[BMFont](https://www.angelcode.com/products/bmfont/doc/file_format.html) - ファイル形式の元祖仕様

[Glyph Designer](https://web.archive.org/web/20160830115758/https://71squared.com/blog/bitmap-font-file-format) - 出力の詳細（バイナリ形式も含む）


## ビットマップ作成ツール

[Hiero](/wiki/tools/hiero) - システムフォントをビットマップに変換するユーティリティ

[ShoeBox](https://renderhjs.net/shoebox/)  - 画像からカスタマイズしたグリフを読み込み、それらからビットマップフォントを作成できます。[libGDXでの使い方の良いチュートリアル動画](https://www.youtube.com/watch?v=dxPf1M7YORU)もあります。

[Glyph Designer](https://www.71squared.com/en/glyphdesigner) - 影、グラデーション、ストロークなど幅広いオプションを備えた商用ビットマップフォントツール。

[Littera](https://kvazars.com/littera) - オンラインのビットマップフォント生成ツール。カスタマイズ項目が豊富です（Adobe Flash が必要）。

## その他のツール

[FreeTypeFontGenerator](https://web.archive.org/web/20200423064636/ttp://www.badlogicgames.com/wordpress/?p=2300) - Hiero のようなユーティリティで事前にレンダリング済みのビットマップを用意する代わりに、フォントからビットマップを生成します

例
: [(さらに見る)](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/extensions/InternationalFontsTest.java)

	FreeTypeFontGenerator generator = new FreeTypeFontGenerator(Gdx.files.internal("data/unbom.ttf"));

	FreeTypeFontParameter parameter = new FreeTypeFontParameter();
	parameter.size = 18;
	parameter.characters = "한국어/조선�?";

	BitmapFont koreanFont = generator.generateFont(parameter);

	parameter.characters = FreeTypeFontGenerator.DEFAULT_CHARS;
	generator = new FreeTypeFontGenerator(Gdx.files.internal("data/russkij.ttf"));
	BitmapFont cyrillicFont = generator.generateFont(parameter);
	generator.dispose();



[Distance field fonts](/wiki/graphics/2d/fonts/distance-field-fonts) - 拡大・回転しても汚いアーティファクトが出にくく、フォントのスケーリングに便利です

[gdx-smart-font](https://github.com/jrenner/gdx-smart-font) - 画面サイズに応じてビットマップフォントを自動生成・キャッシュする非公式のlibGDXアドオン（`FreeTypeFontGenerator`を使用）

## 3D空間内のフォント
libGDXはテキストを3D空間に直接配置する機能を提供していませんが、それでも比較的簡単に実現できます。例として、[このgist](https://gist.github.com/Darkyenus/e9427b0655816d2a521227cb9313d303)を参照してください。なお、`SpriteBatch`は一定の`z`で描画するため、手前にあるオブジェクトによるオクルージョン（隠れ）が発生しません。ただし、適切な`z`をuniformから設定するカスタムシェーダーを用意すれば、修正はそれほど難しくありません。

## 等幅フォント
すべてのグリフを同じ幅で表示しなければならないフォント（等幅フォント）は、少し厄介な問題があります。たとえば`|`のように細い文字の左側にある「初期の空白」は、デフォルトでは表示されません。その結果、その細い文字が直前の文字に“くっつく”ように並び、本来意図した空白が入らなくなります。さらに、その文字の幅が小さいことで、複数行のテキストが互いに揃わないといった問題も起こりえます。これを解決するために、`BitmapFont#setFixedWidthGlyphs(CharSequence)`という既存メソッドがあります。指定した`CharSequence`（通常は`String`）に含まれる文字について、これらの問題をまとめて解消できます。ただし注意点として、「同じ幅にしたいグリフを、フォント内の該当文字すべて列挙する必要がある」ため、大きなフォントだと手間が増えます。ASCIIあるいはその小さな拡張程度で済むなら、HieroにはASCIIやNeHeのボタンがあり、よく使う小さめの文字セットでテキストフィールドを埋められます。それをコピーして、`setFixedWidthGlyphs()`に渡すStringとして使うとよいでしょう。一方、等幅にしたい文字の全リストが非常に多い／未知である等幅フォントの場合は、次のコードで「すべてのグリフを等幅化」できます。最大のグリフ幅を全グリフに適用します。
```java
        public static void setAllFixedWidth(BitmapFont font) {
            BitmapFont.BitmapFontData data = font.getData();
            int maxAdvance = 0;
            for (int index = 0, end = 65536; index < end; index++) {
                BitmapFont.Glyph g = data.getGlyph((char) index);
                if (g != null && g.xadvance > maxAdvance) maxAdvance = g.xadvance;
            }
            for (int index = 0, end = 65536; index < end; index++) {
                BitmapFont.Glyph g = data.getGlyph((char) index);
                if (g == null) continue;
                g.xoffset += (maxAdvance - g.xadvance) / 2;
                g.xadvance = maxAdvance;
                g.kerning = null;
                g.fixedWidth = true;
            }
        }
```
