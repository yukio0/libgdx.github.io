---
title: Bulletラッパーの使い方
---
## Bulletの初期化

Bulletを使う前に、ライブラリをロードする必要があります。これは`create`メソッドに次の1行を追加することで行えます。
```java
Bullet.init();
```

初期化が完了する前にBulletを使わないよう注意してください。例えば次のコードは、ライブラリがロードされる前に`btGhostPairCallback`が生成されてしまうため、エラーになります。
```java
public class InvokeRuntimeExceptionTest {
  final static btGhostPairCallback ghostPairCallback = new btGhostPairCallback();
}
```

## Bulletラッパーの基本
このラッパーは、元のBulletのクラス名にできるだけ従う傾向があります。つまり、多くのクラス名は「bt」で始まります。いくつか例外もありますが、主にネストされた構造体（struct）です。これらは`com.badlogic.gdx.physics.bullet`パッケージ内に、カスタム実装として直接用意されています。残念ながら、一部のネストされた構造体や一部の基底クラスは、1対1での変換に向きません。詳しくは「カスタムクラス」のセクションを参照してください。もしラッパーに存在しないクラスを見つけた場合は、フォーラムまたはIssueトラッカー（https://github.com/libgdx/libgdx/issues）に投稿すれば、ラッパーに追加できるかもしれません。

## コールバック

コールバックは特に注意が必要です。デフォルトでは、ラッパーは一方向（Java → C++）のやり取りのみをサポートします。C++側からJavaコードを呼び出す必要があるコールバック用インターフェースは、個別にカスタム実装されています。まだ実装されていないコールバック用インターフェースを見つけた場合は、フォーラムに投稿すればラッパーへ追加できるかもしれません。

コールバック用インターフェース一覧（完全ではない可能性があります）
 * `LocalShapeInfo`
 * `LocalRayResult`
 * `RayResultCallback`
 * `ClosestRayResultCallback`
 * `AllHitsRayResultCallback`
 * `LocalConvexResult`
 * `ConvexResultCallback`
 * `ClosestConvexResultCallback`
 * `ContactResultCallback`
 * `btMotionState`
 * `btIDebugDraw`
 * `InternalTickCallback`
 * `ContactListener`
 * `ContactCache`

## プロパティ
プロパティは`getter`／`setter`メソッドでカプセル化されています。`getter`／`setter`の命名では、`m_`プレフィックスは省略されます。例えば、ネイティブクラス`btCollisionObjectWrapper`の`m_collisionObject`メンバーは、`getCollisionObject()`と`setCollisionObject(...)`として実装されています。

## オブジェクトの生成と破棄
Java でBulletクラスを生成すると、そのたびに対応するC++側のクラスも生成されます。Java側のオブジェクトはガベージコレクタによって管理されますが、C++側はそうではありません。孤立したC++オブジェクトが残ってメモリリークになるのを防ぐため、デフォルトではJavaオブジェクトがガベージコレクタによって破棄されるときに、C++オブジェクトも自動的に破棄されます。

ただしこれは便利な場合もある一方で、あくまでフェイルセーフ（fail-safe）であり、これに頼るべきではありません。ガベージコレクタは制御できないため、オブジェクトが実際に *破棄されるかどうか*／*いつ破棄されるか*／*どの順序で破棄されるか*を制御できません。そのため、ガベージコレクタによってオブジェクトが自動破棄された場合、ラッパーはエラーをログに出します。このエラーログは`Bullet.init()`の第2引数で無効化できますが、できれば次の段落で説明する方法を使うことを推奨します。

正しく破棄を行うには、生成した各オブジェクトへの参照を「不要になるまで」保持し、不要になったら自分で破棄するべきです。Javaオブジェクトに対して`.dispose()`を呼ぶことでC++側のオブジェクトを破棄できます。`dispose()`後はJavaオブジェクトは使用できないため、参照もすべて外してください。

上記が当てはまるのは、あなたが責任を持つオブジェクトだけです。つまり`new`キーワードで作ったBulletクラスと、ヘルパーメソッドで生成したクラスです。通常のメソッド戻り値として返されるオブジェクトや、コールバックメソッドで渡されるオブジェクトは破棄する必要はありません。

## オブジェクト参照
上で述べたとおり、各Bulletクラスへの参照を保持し、不要になったら`dispose`を呼ぶべきです。しかしアプリケーションが複雑になり、複数のオブジェクト間で共有されるようになると、参照の追跡が難しくなる場合があります。そこでBulletラッパーは参照カウントをサポートしています。

参照カウントはデフォルトでは無効です。有効にするには、`Bullet.init();`を第1引数`true`で呼び出します。
```java
Bullet.init(true);
```

参照カウントを使う場合、参照したい各オブジェクトに対して`obtain()`を必ず呼んでください。参照が不要になったら`release()`を呼びます。`release()`は、そのオブジェクトに他の参照が残っていなければ`dispose`します。

いくつかのラッパークラスは参照管理を助けてくれます。たとえば`btCompoundShape`は子シェイプすべてへの参照を`obtain()`し、自身が`dispose`されるときにそれらを`release()`します。

## クラスの拡張
Bulletのクラスは拡張できますが、コールバック用クラスを除いて基本的には推奨されません（コールバックの場合も、意図されたメソッドだけをオーバーライドしてください）。追加した情報はC++側には伝わりません。また、Bulletラッパーのメソッドが「あなたが拡張したクラス」を返すこともありません。
```java
btCollisionShape shape = collisionObjectA.getCollisionShape();
```

これは新しいJavaの`btCollisionShape`を生成しますが、あなたが拡張したクラスは実装されません。

例外として`btCollisionObject`だけは、ラッパーが同じJavaクラスを再利用しようとします。さらにJava実装の`btCollisionObject`には、追加データを紐づけるための`userData`メンバーが追加されています。これを実現するため、ラッパーはすべての`btCollisionObject`インスタンス参照を配列で保持します。この配列には静的フィールド`btCollisionObject.instances`からアクセスできます。詳しくは`btCollisionObject`の「./Bullet Wrapper: Custom classes#btcollisionobject」セクションを参照してください。

upcast用メソッドは[このissue](https://code.google.com/archive/p/libgdx/issues/1453)のため提供されていません。Javaで生成したクラスについては、これらは不要です。これらのクラスは直接キャストできます。

## クラスの比較
ラッパークラスは`equals()`メソッドで比較できます。これは、両者が同じネイティブクラスをラップしているかどうかをチェックします。基になるC++クラスへのポインタは、対象オブジェクトの`getCPointer`メソッドで取得できます。これらのポインタを比較することで、Java側のクラスが同じC++クラスをラップしているかどうかを確認することもできます。

## 共通クラス
BulletはlibGDX coreにも存在するクラスをいくつか使います。これらのBulletクラスは利用可能ですが、ラッパーは可能な限りlibGDX側のクラスを使うようにしています。現在は次の対応が実装されています。

| *Bullet* | *Libgdx* |
|:--------:|:--------:|
| btVector3 | Vector3 |
| btQuaternion | Quaternion |
| btMatrix3x3 | Matrix3 |
| btTransform | Matrix4 |
| btScalar | float |

<sub>`Matrix4`から`btTransform`への変換では、`btTransform`が`origin`と`rotation`しか持たないため、一部情報が失われる可能性があります。また、`btScalar`はプリミティブ型`float`のシノニムです。</sub>

これら共通クラス用のオブジェクト生成を避けるため、ラッパーは同じインスタンスを再利用します。したがって、次の2点に注意してください。

 1. こうしたクラスを返すラッパーメソッドの戻り値は、同じ型を返す次の呼び出しによって上書きされます。
```java
// 間違ったケース
Matrix4 transformA = collisionObjectA.getWorldTransform();
// transformAはcollisionObjectAのworldTransformを保持している
Matrix4 transformB = collisionObjectB.getWorldTransform();
// transformAとtransformBは同一オブジェクトで、いまはcollisionObjectBのworldTransformを保持している

// 正しいケース
transformA.set(collisionObjectA.getWorldTransform());
transformB.set(collisionObjectB.getWorldTransform());
```
 2. こうしたクラスを引数として受け取るインターフェースコールバックの引数は、呼び出し後には使用できません。
```java
// 間違ったケース
@Override
public void setWorldTransform (final Matrix4 worldTrans) {
	transform = worldTrans;
}
// 正しいケース
@Override
public void setWorldTransform (final Matrix4 worldTrans) {
	transform.set(worldTrans);
}
```

## 配列を使う
可能な場合、ラッパーはJavaからC++へ配列を渡すのにダイレクト`ByteBuffer`を使います。これにより呼び出し時の配列コピーを避けられ、OpenGL ESとBulletの両方で同じバイトバッファを共有できます。必要なら `BufferUtils.newUnsafeByteBuffer`で新しい`ByteBuffer`を作成し、`BufferUtils.disposeUnsafeByteBuffer`で手動で削除してください。

`ByteBuffer`が使えない、または使いたくない場合は通常の配列が使われます。デフォルトでは、配列はメソッド開始時にJavaからC++へ反復コピーされ、メソッド終了時にC++からJavaへコピーされます。このオーバーヘッドを避けるため、ラッパーは可能な場合、JNIのクリティカル配列（critical arrays）を使ってC++側からJava配列を直接利用しようとします。この方法を使うメソッドの実行中はJavaのガベージコレクションがブロックされます。こうしたメソッドの例が`btBroadphasePairArray.getCollisionObjects`です。
