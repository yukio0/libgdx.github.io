---
title: Bulletラッパーの接触コールバック
---
接触コールバック（contact callbacks）を使うと、2 つのオブジェクト間で接触が発生したときに通知を受けられます（[詳細とパフォーマンス上の注意](https://web.archive.org/web/20180906172418/http://bulletphysics.org/mediawiki-1.5.8/index.php/Collision_Callbacks_and_Triggers)）。

デフォルトでは3つのコールバックがあります：[`onContactAdded`](https://web.archive.org/web/20180906172418/http://bulletphysics.org/mediawiki-1.5.8/index.php/Collision_Callbacks_and_Triggers#gContactAddedCallback)、[`onContactProcessed`](https://web.archive.org/web/20180906172418/http://bulletphysics.org/mediawiki-1.5.8/index.php/Collision_Callbacks_and_Triggers#gContactProcessedCallback)、そして [`onContactDestroyed`](https://web.archive.org/web/20180906172418/http://bulletphysics.org/mediawiki-1.5.8/index.php/Collision_Callbacks_and_Triggers#gContactDestroyedCallback)です。ラッパーはさらに2つのコールバック`onContactStarted`と`onContactEnded`を追加します（[詳細](https://pybullet.org/Bullet/phpBB3/viewtopic.php?t=7739&p=32470)）。これらのコールバックはグローバル（例：`CollisionWorld`とは独立）であり、同時に有効化できる実装は各コールバックにつき 1 つだけです。

### 接触リスナー（Contact Listener）
`ContactListener`クラスを拡張して、1つ以上のコールバックを実装できます。
```java
public class MyContactListener extends ContactListener {
	@Override
	public void onContactStarted (btCollisionObject colObj0, btCollisionObject colObj1) {
		// 実装する
	}
	@Override
	public void onContactProcessed (int userValue0, int userValue1) {
		// 実装する
	}
}
```

各コールバックにつき、有効にできるリスナーは同時に1つだけです。`enable();`メソッドを使うと、そのリスナーをアクティブにし、同じコールバック上の他のリスナーは無効化されます。`disable();`メソッドを使うと、そのコールバックの通知を受け取らなくなります。リスナーをインスタンス化すると自動的にそのコールバックは有効化され、破棄（`dispose();`メソッド）すると自動的に無効化されます。

`ContactListener`クラスは、各コールバックに対してオーバーライドできるメソッドシグネチャを複数提供しています。たとえば`onContactAdded`コールバックは、次のシグネチャでオーバーライドできます。

```java
boolean onContactAdded(btManifoldPoint cp, btCollisionObjectWrapper colObj0Wrap, int partId0, int index0, btCollisionObjectWrapper colObj1Wrap, int partId1, int index1);

boolean onContactAdded(btManifoldPoint cp, btCollisionObject colObj0, int partId0, int index0, btCollisionObject colObj1, int partId1, int index1);

boolean onContactAdded(btManifoldPoint cp, int userValue0, int partId0, int index0, int userValue1, int partId1, int index1);

boolean onContactAdded(btCollisionObjectWrapper colObj0Wrap, int partId0, int index0, btCollisionObjectWrapper colObj1Wrap, int partId1, int index1);

boolean onContactAdded(btCollisionObject colObj0, int partId0, int index0, btCollisionObject colObj1, int partId1, int index1);

boolean onContactAdded(int userValue0, int partId0, int index0, int userValue1, int partId1, int index1);
```

見てのとおり、`btManifoldPoint`を受け取るものが3つ、受け取らないものが3つあります。実際の衝突オブジェクトを受け取る方法は、`btCollisionObjectWrapper`、`btCollisionObject`、または`userValue`のいずれかを選べます。

実際に使う引数だけが渡されるメソッドをオーバーライドするようにしてください。たとえば`btManifoldPoint`を使わないのであれば、コールバックが呼ばれるたびにその引数用のオブジェクトを生成するのは無駄になります。同様に、`btCollisionObject`は再利用されるため、`btCollisionObjectWrapper`を使うよりも高パフォーマンスです。`userValue`はそもそもオブジェクトのマッピングを行わないため、さらに高パフォーマンスになります（`userValue`の使い方は[#btCollisionObject btCollisionObject]を参照してください）。

`onContactAdded`コールバックは、衝突した2つのボディの少なくとも一方に`CF_CUSTOM_MATERIAL_CALLBACK`が設定されている場合にのみ呼び出されます。
```java
body.setCollisionFlags(e.body.getCollisionFlags() | btCollisionObject.CollisionFlags.CF_CUSTOM_MATERIAL_CALLBACK);
```

added／processed／destroyedの各コールバック間で同一の接触を識別したい場合は、コールバックが提供する`btManifoldPoint`インスタンスの`setUserValue(int);`と`getUserValue();`を使えます。この値は`ContactListener`クラスの`onContactDestroyed(int)`メソッドにも渡されます。なお`onContactDestroyed`コールバックは、`userValue`が0以外のときにのみ呼び出されます。

### 接触フィルタリング（Contact Filtering）

接触コールバックは非常に頻繁に呼び出されます。呼び出しのたびにC++とJava 間でJNIブリッジを行うとオーバーヘッドが大きく、パフォーマンスが低下します。そこでBulletラッパーでは、「どのオブジェクトの接触通知を受け取るか」を指定できるようになっています。これが接触フィルタリングです。

衝突フィルタリングと同様に、各`btCollisionObject`に対して`setContactCallbackFlag(int);`でフラグを、`setContactCallbackFilter(int);`でフィルタを指定できます。オブジェクトAのフィルタがオブジェクト Bにマッチする条件は`A.filter & B.flag == B.flag`です。一方または両方のフィルタがマッチした場合にのみ、その接触がリスナーに渡されます。

```java
static int PLAYER_FLAG = 2; // 2番目のビット
static int COIN_FLAG = 4; // 3番目のビット
btCollisionObject player;
btCollisionObject coin;
...
player.setContactCallbackFlag(PLAYER_FLAG);
coin.setContactCallbackFilter(PLAYER_FLAG);
// コインがプレイヤーと接触した場合にのみリスナーが呼び出される
```

デフォルトでは、`btCollisionObject`の`contactCallbackFlag`は1、`contactCallbackFilter`は0に設定されています。なおフラグを0に設定すると、そのオブジェクトについては常にコールバックが呼ばれるようになる点に注意してください（`x & 0 == 0`となるため）。

接触フィルタリングを使うかどうかは、どのメソッドシグネチャをオーバーライドするかで決まります。接触フィルタリングに対応した各コールバックについて、`ContactListener`クラスは`boolean match0`と`boolean match1`引数を含むシグネチャを提供しています。これをオーバーライドした場合、そのメソッドでは接触フィルタリングが使用されます。`boolean match`引数を持たないシグネチャをオーバーライドした場合、そのメソッドでは接触フィルタリングは使用されません。

`boolean match0`と`boolean match1`の値を使って、どちらのフィルタがマッチしたのかを確認できます。
```java
public class MyContactListener extends ContactListener {
	@Override
	public void onContactEnded (int userValue0, boolean match0, int userValue1, boolean match1) {
		if (match0) {
			// オブジェクト0（userValue0）がマッチ
		}
		if (match1) {
			// オブジェクト1（userValue1）がマッチ
		}
	}
}
```

接触フィルタリングを使っていても、衝突時にはコールバックがかなり頻繁に呼ばれる場合があります。これを避けるには、処理後にフィルタを0に設定してしまう方法があります。たとえば「プレイヤーとコイン」のケースでは、プレイヤーがコインに当たり続けるのを許すのではなく、コインがプレイヤーに当たった最初の接触でコイン側のフィルタを0にするのが最適です。
