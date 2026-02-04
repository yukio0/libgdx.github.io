---
title: ライフサイクル
---
libGDXアプリケーションには明確に定義されたライフサイクルがあり、アプリケーションの状態（作成、ポーズと再開、描画、破棄など）を制御します。

## ApplicationListener
アプリケーション開発者は、[ApplicationListener](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/ApplicationListener.html)インターフェースを実装し、実装のインスタンスを特定バックエンドの`Application`実装に渡すことで、これらのライフサイクルイベントにフックします（[アプリケーションフレームワーク](/wiki/app/the-application-framework)参照）。その後、アプリケーションレベルのイベントが発生するたびに、`Application`が`ApplicationListener`を呼び出します。最小構成の`ApplicationListener`実装は次のようになります。

```java
public class MyGame implements ApplicationListener {
   public void create () {
   }

   public void render () {        
   }

   public void resize (int width, int height) {
   }

   public void pause () {
   }

   public void resume () {
   }

   public void dispose () {
   }
}
```

また、[ApplicationAdapter](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/ApplicationAdapter.html)クラスを継承する方法もあります。この場合は、これらのメソッドに対して空のデフォルト実装を提供します。

`Application`に渡された後、`ApplicationListener`の各メソッドは次のように呼び出されます。

| Method signature | Description |
| ---------------- | ----------- |
| `create ()` | アプリケーションが作成されたときに一度だけ呼ばれます。 |
| `resize (int width, int height)` | ゲーム画面がリサイズされ、かつゲームがポーズ状態ではないときに呼ばれます。また`create()`の直後にも一度呼ばれます。<br/>引数は、リサイズ後の画面の幅・高さ（ピクセル）です。 |
| `render ()` | 描画が行われるべきタイミングごとに、アプリケーションのゲームループから呼ばれます。ゲームロジックの更新も通常はこのメソッド内で行います。 |
| `pause ()` | AndroidではHomeボタンが押されたときや着信があったときに呼ばれます。デスクトップではウィンドウが最小化されたとき、そしてアプリ終了時に`dispose()`の直前にも呼ばれます。<br/>ゲーム状態を保存するのに適した場所です。 |
| `resume ()` | Androidではポーズ状態から復帰したときに呼ばれ、デスクトップでは最小化解除時に呼ばれます。 |
| `dispose ()` | アプリケーションが破棄されるときに呼ばれます。呼び出しの前に`pause()`が呼ばれます。 |

次の図はライフサイクルを視覚的に示したものです。

![images/70efff32-dd28-11e3-9fc4-1eb57143aee6.png](/assets/wiki/images/70efff32-dd28-11e3-9fc4-1eb57143aee6.png)

## メインループはどこ？
libGDXは本質的にイベント駆動型で、これは主にAndroidやJavaScriptの動作方式に由来します。明示的なメインループは存在しませんが、`ApplicationListener.render()`メソッドをそのメインループの本体とみなすことができます。

## 参考
Androidを対象にするなら、[libGDX and Android lifecycle](https://bitiotic.com/blog/2013/05/23/libgdx-and-android-application-lifecycle/)が参考になります。この記事では、なぜstatic変数を使うべきではないのかも説明されています。
