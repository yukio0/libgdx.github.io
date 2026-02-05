---
title: スレッド
---
`ApplicationListener`の各メソッドは、すべて同じスレッド上で呼び出されます。このスレッドはレンダリングスレッドであり、OpenGLの呼び出しを行えるスレッドです。多くのゲームでは、ロジック更新と描画の両方を`ApplicationListener.render()`メソッド内（＝レンダリングスレッド上）で実装すれば十分です。

OpenGLに直接関わるグラフィックス処理は、必ずレンダリングスレッドで実行する必要があります。別スレッドで実行すると、動作は未定義になります。これはOpenGLコンテキストがレンダリングスレッド上でのみ有効だからです。OpenGLコンテキストを別スレッドでカレントにすることは、多くのAndroid端末で問題が起きるため、これはサポートされていません。

別スレッドからレンダリングスレッドへデータを渡すには、[`Application.postRunnable()`](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/Application.java#L188)を使うことを推奨します。これにより、`Runnable`内のコードが次のフレームでレンダリングスレッド上で実行され、`ApplicationListener.render()`が呼ばれる前に処理されます。

```java
new Thread(new Runnable() {
   @Override
   public void run() {
      // レンダリングスレッドとは非同期に、ここで重要な処理を行う
      final Result result = createResult();
      // 結果を処理するRunnableをレンダリングスレッドへ投げる
      Gdx.app.postRunnable(new Runnable() {
         @Override
         public void run() {
            // 結果を処理する（例：ApplicationListenerのArray<Result>フィールドに追加する）
            results.add(result);
         }
      });
   }
}).start();
```

## libGDX のクラスはどれがスレッドセーフですか？
libGDXのクラスは、クラスドキュメントで**明示的に**スレッドセーフと書かれていない限り、スレッドセーフではありません！

特に、グラフィックスやオーディオに関係するものに対して、マルチスレッドで操作を行うべきではありません。例えば、scene2Dのコンポーネントを複数スレッドから触る、といった使い方は避けてください。

## HTML5
JavaScriptは本質的にシングルスレッドです。そのため、[HTML5バックエンドの制約](/wiki/html5-backend-and-gwt-specifics#differences-between-gwt-and-desktop-java)の1つとして、スレッディングは利用できません。将来的には[Web Workers](https://html.spec.whatwg.org/multipage/workers.html)が選択肢になるかもしれませんが、スレッド間のデータ受け渡しはメッセージパッシングで行われます。Javaのスレッドとは考え方や仕組みが違うので、マルチスレッドのコードをWeb Workersに持っていくのは一筋縄ではいきません。
