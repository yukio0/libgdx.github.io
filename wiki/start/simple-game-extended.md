---
title: "シンプルなゲームの拡張"
redirect_from:
  - /dev/simple-game-extended/
  - /dev/simple_game_extended/
---

このチュートリアルでは、[前回のチュートリアル](/wiki/start/a-simple-game)で作った**シンプルなゲーム**「Drop」を**拡張**していきます。今回は、メニュー画面やいくつかの新機能を追加して、ゲームを少し充実させていきます。

では、libGDXの少し高度なクラスについて紹介していきましょう。

## スクリーンインターフェース
複数の要素を持つゲームにおいて、スクリーンは_基本的な_存在です。スクリーンは`ApplicationListener`オブジェクトで慣れ親しんだメソッドの多くを備えており、さらにいくつかの新しいメソッドも持っています。新しいメソッドである`show`と`hide`はスクリーンがフォーカスされたとき、または失ったときにそれぞれ呼ばれます。スクリーンはゲームの一側面（例えば、メニュー画面、設定画面、ゲーム画面など）の処理や描画を担当します。

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
Now that we have our main menu finished, it's time to finally get to making our game. We will be lifting most of the code from the [original game](/wiki/start/a-simple-game) as to avoid redundancy, and avoid having to think of a different game idea to implement as simply as Drop is.


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

		// load the images for the background, bucket and droplet
		backgroundTexture = new Texture("background.png");
		bucketTexture = new Texture("bucket.png");
		dropTexture = new Texture("drop.png");

		// load the drop sound effect and background music
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
		// start the playback of the background music
		// when the screen is shown
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

This code is almost 95% the same as the original implementation, except now we use a constructor instead of the `create()` method of the `ApplicationListener`, and pass in a `Drop` object, like in the `MainMenuScreen` class. We also start playing the music as soon as the Screen is set to `GameScreen`. Moreover, we added a string to the top left corner of the game, which tracks the number of raindrops collected.

Note that the `dispose()` method of the `GameScreen` class is not called automatically, see the [Screen API](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Screen.html). It is your responsibility to take care of that. You can call this method from the `dispose()` method of the `Game` class, if the `GameScreen` class passes a reference to itself to the `Game` class or by calling `screen.dispose()` in `Drop` class `dispose()` method. It is important to do this, else `GameScreen` assets might persist and occupy memory even after exiting the application.

And that's it, you have the complete game finished. That is all there is to know about the Screen interface and abstract Game Class, and all there is to creating multifaceted games with multiple states. The **full Java code** can be found [here](https://github.com/libgdx/libgdx.github.io/tree/dev/assets/downloads/tutorials/extended-game-java). If you are developing in **Kotlin**, take a look [here](https://github.com/libgdx/libgdx.github.io/tree/dev/assets/downloads/tutorials/extended-game-kotlin) for the full code.

## 今後の展望
After this tutorial you should have a basic understanding how libGDX works and what to expect going forward. Some things can still be improved, like using the [Memory Management](/wiki/articles/memory-management#object-pooling) classes to recycle all the Rectangles we have the garbage collector clean up each time we delete a raindrop. OpenGL is also not too fond if we hand it too many different images in a batch (in our case it's OK as we only had two images). Usually one would put all those images into a single `Texture`, also known as a `TextureAtlas`. In addition, taking a look at [Viewports](/wiki/graphics/viewports) will most certainly prove useful. Viewports help dealing with different screen sizes/resolutions and decide, whether the screen's content needs to be stretched/should keep its aspect ratio, etc.

To continue learning about libGDX we highly **recommend reading our [wiki](/wiki/)** and checking out the demos and tests in our main GitHub repository. If you have any questions, **join our official [Discord server](/community/)**, we are always glad to help!

The best practice is to get out there and do it, so farewell and happy coding!
