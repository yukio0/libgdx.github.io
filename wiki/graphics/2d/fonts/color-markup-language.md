---
title: カラーマークアップ言語
---
`BitmapFontCache`クラスは、簡単なマークアップ言語を使って、文字列中で色付きテキストを扱えます。

マークアップはデフォルトでは無効です。publicメンバーである[font.getData().markupEnabled](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/graphics/g2d/BitmapFont.BitmapFontData.html#markupEnabled)を使って、有効／無効を切り替えてください。

マークアップ構文はとてもシンプルですが、柔軟に使えます。
- **[name]** 名前で色を指定します。いくつかの色はあらかじめ定義されています。完全な一覧は[Colors.reset()](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/graphics/Colors.java)メソッドを参照してください。ユーザーは[Colors](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/graphics/Colors.html)クラスのメソッドを使って独自の色を定義することもできます。
- **[#xxxxxxxx]** 16進値`xxxxxxxx`で色を指定します。形式は`RRGGBBAA`で、AA（アルファ値）は省略可能です。省略した場合は`0xFF`になります。
- **[]** 色を「直前の色」に戻します（任意の終了タグのようなものです）。
- **[[** 左角括弧`[`をエスケープします。

色名は大文字・小文字を区別します。また、空文字は不可で、先頭に`#`や`[` は使えず、`]`を含めることもできません。さらに、色名の中に`[`が含まれる場合、その`[`はエスケープしてはいけません。

不明な色名、不正な16進コード、閉じられていないタグは、警告などは出ずに静かに無視され、通常のテキストとして扱われます。

サンプルコードはテストクラス[BitmapFontTest](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/BitmapFontTest.java)を参照してください。

**注:** Scene2Dで使用する場合、Labelでマークアップの色指定を有効にするには、**skin.json**の**LabelStyle**定義から**fontColor**プロパティを削除する必要があります。