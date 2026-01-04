---
title: "シンプルなゲームの拡張"
redirect_from:
  - /dev/simple-game-extended/
  - /dev/simple_game_extended/
---

このチュートリアルでは、[前回のチュートリアル](/wiki/start/a-simple-game)で作った**シンプルなゲーム**「Drop」を**拡張**していきます。今回は、メニュー画面やいくつかの新機能を追加して、ゲームを少し充実させていきます。

では、libGDXの少し高度なクラスについて紹介していきましょう。

## スクリーンインターフェース
複数の要素を持つゲームにおいて、スクリーンは _基本的な_ 存在です。スクリーンは`ApplicationListener`オブジェクトで慣れ親しんだメソッドの多くを備えており、さらにいくつかの新しいメソッドも持っています。新しいメソッドである`show`と`hide`はスクリーンがフォーカスされたとき、または失ったときにそれぞれ呼ばれます。スクリーンはゲームの一側面（例えば、メニュー画面、設定画面、ゲーム画面など）の処理や描画を担当します。

## ゲームクラス
`Game`クラスは複数のスクリーンを管理する役割を持ち、そのためのヘルパーメソッドをいくつか提供するとともに、`ApplicationListener`の実装も備えています。`Screen`と`Game` オブジェクトを組み合わせることで、シンプルながら強力なゲーム構造を作ることができます。

ここからは、`Game`クラスを継承した`Drop`クラスを作成し、その`create()`メソッドをゲームのエントリーポイントとします。コードを見てみましょう。

```java
package com.badlogic.drop;

import com.badlogic.gdx.Game;
import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.utils.viewport.FitViewport;


public class Drop extends Game {

	public SpriteBatch batch;
	public BitmapFont font;
	public FitViewport viewport;


	public void create() {
		batch = new SpriteBatch();
		// libGDXのデフォルトフォントを使用します
		font = new BitmapFont();
		viewport = new FitViewport(8, 5);
		
		// フォントサイズは15ptですが、ビューポートの高さと画面の高さの比率でスケーリングする必要があります
		font.setUseIntegerPositions(false);
		font.getData().setScale(viewport.getWorldHeight() / Gdx.graphics.getHeight());
		
		this.setScreen(new MainMenuScreen(this));
	}

	public void render() {
		super.render(); // 重要！
	}

	public void dispose() {
		batch.dispose();
		font.dispose();
	}

}
```

アプリケーションはスプライトバッチ、ビットマップフォント、ビューポートをインスタンス化して開始します。共有できるオブジェクトを何度も作成するのは望ましくない方法です（[DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)を参照）。スプライトバッチはテクスチャなどのオブジェクトを画面に描画するために使用されます。ビットマップフォントはスプライトバッチと組み合わせて、画面に文字を描画するために使われます。これについては、後ほどスクリーンインターフェースを実装したクラスの説明で詳しく触れます。

次に、ゲームのスクリーンを`MainMenuScreen`オブジェクトに設定します。このとき、Dropインスタンスを唯一の引数として渡します。

よくあるミスは`Game`を継承したクラスで`super.render()`を呼び忘れることです。この呼び出しがないと、`Game`を継承したクラス内で`render`メソッドをオーバーライドした場合、`create()`メソッドで設定したスクリーンが描画されません！
{: .notice--primary}

最後にもう一つ、重いオブジェクトは必ず破棄（dispose）することを忘れないでください！詳しい解説は[こちら](/wiki/managing-your-assets)で確認してください。


## メインメニュー
それでは、`MainMenuScreen`クラスの核心部分に踏み込んでみましょう。

```java
package com.badlogic.drop;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Screen;

public class MainMenuScreen implements Screen {

	final Drop game;

	public MainMenuScreen(final Drop game) {
		this.game = game;
	}


        //...残りのクラス部分は省略します

}
```

このコード例では、`Screen`インターフェースを実装する`MainMenuScreen`クラスのコンストラクタを作成しています。`Screen`インターフェースには`create()`メソッドのようなメソッドは用意されていないため、代わりにコンストラクタを使用します。今回のゲームでは、コンストラクタに必要なのは`Drop`のインスタンスだけです。これを渡すことによって、必要に応じて`Drop`インスタンスのメソッドやフィールドを呼び出すことが出来ます。

次に、`MainMenuScreen`クラスの最後の「重要な」メソッド`render(float)`メソッドです。

```java
public class MainMenuScreen implements Screen {

        //public MainMenuScreen(final Drop game)....

	@Override
	public void render(float delta) {
		ScreenUtils.clear(Color.BLACK);

		game.viewport.apply();
		game.batch.setProjectionMatrix(game.viewport.getCamera().combined);

		game.batch.begin();
		// テキストを描画します。xとyはメートル単位であることを忘れないでください。
		game.font.draw(game.batch, "Welcome to Drop!!! ", 1, 1.5f);
		game.font.draw(game.batch, "Tap anywhere to begin!", 1, 1);
		game.batch.end();

		if (Gdx.input.isTouched()) {
			game.setScreen(new GameScreen(game));
			dispose();
		}
	}

        // 残りのクラス部分は省略します...

}

```

ここでのコードは比較的単純ですが、独自にスプライトバッチやビットマップフォントを作るのではなく、`game`が持つインスタンスを使う点に注意してください。テキストを画面に描画する方法は`game.font.draw(SpriteBatch, String, float, float)`です。libGDXにはあらかじめ用意されたフォント（Arial）が含まれているので、デフォルトコンストラクタを使うだけでもフォントを利用できます。

次に、画面がタッチされたかどうかを確認します。もしタッチされていれば、ゲームの画面を`GameScreen`インスタンスに切り替え、現在の`MainMenuScreen`のインスタンスを破棄します。`MainMenuScreen`で実装する必要のある他のメソッドは空のままにしてあるので、ここでも省略します（このクラスでは破棄すべきものは特にありません）。

また、リサイズ時にビューポートを更新することを忘れないでください。

```java
@Override
public void resize(int width, int height) {
	game.viewport.update(width, height, true);
}
```

## ゲームスクリーン
メインメニューが完成したので、いよいよゲーム本編を作る時です。重複する作業を避け、Dropのようにシンプルに実装できる別のゲームアイデアを考える手間を省くため、ほとんどのコードは[前回作ったゲーム](/wiki/start/a-simple-game)から流用します。


```java
package com.badlogic.drop;


import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.Screen;
import com.badlogic.gdx.audio.Music;
import com.badlogic.gdx.audio.Sound;
import com.badlogic.gdx.graphics.Color;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.Sprite;
import com.badlogic.gdx.math.MathUtils;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Vector2;
import com.badlogic.gdx.utils.Array;
import com.badlogic.gdx.utils.ScreenUtils;

public class GameScreen implements Screen {
	final Drop game;

	Texture backgroundTexture;
	Texture bucketTexture;
	Texture dropTexture;
	Sound dropSound;
	Music music;
	Sprite bucketSprite;
	Vector2 touchPos;
	Array<Sprite> dropSprites;
	float dropTimer;
	Rectangle bucketRectangle;
	Rectangle dropRectangle;
	int dropsGathered;

	public GameScreen(final Drop game) {
		this.game = game;

		// 背景、バケツ、雨粒の画像を読み込みます
		backgroundTexture = new Texture("background.png");
		bucketTexture = new Texture("bucket.png");
		dropTexture = new Texture("drop.png");

		// 雨粒の効果音とBGMを読み込む
		dropSound = Gdx.audio.newSound(Gdx.files.internal("drop.mp3"));
		music = Gdx.audio.newMusic(Gdx.files.internal("music.mp3"));
		music.setLooping(true);
		music.setVolume(0.5F);

		bucketSprite = new Sprite(bucketTexture);
		bucketSprite.setSize(1, 1);
		
		touchPos = new Vector2();
		
		bucketRectangle = new Rectangle();
		dropRectangle = new Rectangle();
		
		dropSprites = new Array<>();
	}

	@Override
	public void show() {
		// スクリーンが表示されたときにBGMの再生を開始
		music.play();
	}

	@Override
	public void render(float delta) {
		input();
		logic();
		draw();
	}

	private void input() {
		float speed = 4f;
		float delta = Gdx.graphics.getDeltaTime();
        
		if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
			bucketSprite.translateX(speed * delta);
		}
		else if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
			bucketSprite.translateX(-speed * delta);
		}
	
		if (Gdx.input.isTouched()) {
			touchPos.set(Gdx.input.getX(), Gdx.input.getY());
			game.viewport.unproject(touchPos);
			bucketSprite.setCenterX(touchPos.x);
		}
	}

	private void logic() {
		float worldWidth = game.viewport.getWorldWidth();
		float worldHeight = game.viewport.getWorldHeight();
		float bucketWidth = bucketSprite.getWidth();
		float bucketHeight = bucketSprite.getHeight();
		float delta = Gdx.graphics.getDeltaTime();
	
		bucketSprite.setX(MathUtils.clamp(bucketSprite.getX(), 0, worldWidth - bucketWidth));
		bucketRectangle.set(bucketSprite.getX(), bucketSprite.getY(), bucketWidth, bucketHeight);
	
		for (int i = dropSprites.size - 1; i >= 0; i--) {
			Sprite dropSprite = dropSprites.get(i);
			float dropWidth = dropSprite.getWidth();
			float dropHeight = dropSprite.getHeight();

            dropSprite.translateY(-2f * delta);
			dropRectangle.set(dropSprite.getX(), dropSprite.getY(), dropWidth, dropHeight);
	
			if (dropSprite.getY() < -dropHeight) dropSprites.removeIndex(i);
			else if (bucketRectangle.overlaps(dropRectangle)) {
				dropsGathered++;
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
		game.viewport.apply();
		game.batch.setProjectionMatrix(game.viewport.getCamera().combined);
		game.batch.begin();
		
		float worldWidth = game.viewport.getWorldWidth();
		float worldHeight = game.viewport.getWorldHeight();
        
		game.batch.draw(backgroundTexture, 0, 0, worldWidth, worldHeight);
		bucketSprite.draw(game.batch);
	
		game.font.draw(game.batch, "Drops collected: " + dropsGathered, 0, worldHeight);
	
		for (Sprite dropSprite : dropSprites) {
			dropSprite.draw(game.batch);
		}
	
		game.batch.end();
	}

	private void createDroplet() {
		float dropWidth = 1;
		float dropHeight = 1;
		float worldWidth = game.viewport.getWorldWidth();
		float worldHeight = game.viewport.getWorldHeight();
	
		Sprite dropSprite = new Sprite(dropTexture);
		dropSprite.setSize(dropWidth, dropHeight);
		dropSprite.setX(MathUtils.random(0F, worldWidth - dropWidth));
		dropSprite.setY(worldHeight);
		dropSprites.add(dropSprite);
	}

	@Override
	public void resize(int width, int height) {
		game.viewport.update(width, height, true);
	}

	@Override
	public void hide() {
	}

	@Override
	public void pause() {
	}

	@Override
	public void resume() {
	}

	@Override
	public void dispose() {
		backgroundTexture.dispose();
		dropSound.dispose();
		music.dispose();
		dropTexture.dispose();
		bucketTexture.dispose();
	}
}
```

このコードは元の実装と95％同じですが、違いとして、`ApplicationListener`の`create()`メソッドの代わりにコンストラクタを使い、`MainMenuScreen`クラスと同じように`Drop`オブジェクトを渡す形になっている点があります。また、スクリーンが`GameScreen`に切り替わった瞬間に音楽の再生を開始するようにしています。さらに、ゲームの左上に集めた雨粒の数を表示する文字列を追加しました。

`GameScreen`クラスの`dispose()`メソッドは自動で呼ばれないことに注意してください。詳しくは[Screen API](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Screen.html)を参照してください。`dispose()`メソッドを呼ぶ責任は開発者側にあります。`GameScreen`クラスの`dispose()`メソッドを呼ぶ方法としては、`GameScreen`クラスが自分自身の参照を`Game`クラスに渡している場合、`Game`クラスの`dispose()`メソッドから呼び出すことができます。あるいは`Drop`クラスの`dispose()`メソッド内で`screen.dispose()`を呼ぶことも可能です。`dispose()`を実行しないと、`GameScreen`のアセットが解放されず、アプリケーションを終了した後もメモリに残ってしまう可能性があります。

これで、ゲームの完成です。これまでで、スクリーンインターフェースとゲーム抽象クラスについて知っておくべきことはすべて説明しました。また、複数の状態を持つ多機能なゲームを作る方法も理解できました。**Java版の完全なコード**は[こちら](https://github.com/libgdx/libgdx.github.io/tree/dev/assets/downloads/tutorials/extended-game-java)で確認できます。**Kotlin**で開発する場合は、[こちら](https://github.com/libgdx/libgdx.github.io/tree/dev/assets/downloads/tutorials/extended-game-kotlin)に完全なコードがあります。

## 今後の展望
このチュートリアルを終えることで、libGDXがどのように動作するか、今後どのようなことができるのかについて基本的な理解が得られるはずです。まだ改善できる点もいくつかあります。例えば、[メモリ管理](/wiki/articles/memory-management#object-pooling)クラスを使って、雨粒を削除するたびにガベージコレクタに任せるのではなく、長方形オブジェクトを再利用する方法があります。また、OpenGLは一度に大量の異なる画像を扱うのが苦手です（今回の例では画像が2枚しかないので問題ありません）。通常、複数の画像を1つのテクスチャにまとめる方法が取られます。これをテクスチャアトラスと呼びます。さらに、[ビューポート](/wiki/graphics/viewports)を確認することも非常に有用です。ビューポートは、画面サイズや解像度の違いに対応するのに役立ち、画面の内容を引き伸ばすか、アスペクト比を維持するかなどを決定するために使います。

libGDX の学習をさらに進めたい場合は、**ぜひ[wiki](/wiki/)を読み**、GitHubリポジトリにあるデモやテストも確認することを強くおすすめします。質問があれば、**公式の[Discordサーバ](https://libgdx.com/community/)に参加してください**。いつでも喜んでサポートします！

最も良い学び方は、実際に手を動かして作ってみることです。では、楽しいコーディングを！
