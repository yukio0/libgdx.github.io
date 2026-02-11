---
title: Bulletラッパーのカスタムクラス
---
場合によっては、C++のBulletクラス／メソッドをJavaのクラス／メソッドとしてそのままラップできないことがあります。そのようなときは、両者を橋渡しするためにカスタムクラスやカスタムメソッドが使われます。以下はそれらの一覧です。なお、このリストは完全ではない可能性があります。

### btCollisionObject

`btCollisionObject`は、毎回新しいJavaオブジェクトを生成するのではなく、Javaオブジェクトを再利用するように改修されています。これは静的な`btCollisionObject.instances`マップを使って行われます。マップからオブジェクトを取り除き、ネイティブ側のオブジェクトを削除するには`dispose`メソッドを使います。

インスタンスの再利用に加えて、Bulletラッパーではインスタンスを識別するための一意な番号を付けられます。たとえば、エンティティシステムにおけるエンティティのインデックス/IDなどです。頻繁に呼ばれるメソッドの中には、インスタンスそのものの代わりにその値を使えるものがあります。これにより、C++とJavaのインスタンスをマッピングするオーバーヘッドを完全に排除できます。この値は`setUserValue(int);`で設定し、`getUserValue();`で取得できます。

```java
public class MyGameObject {
  public btCollisionObject body;
}
...
Array<MyGameObject> gameObjects;
...
gameObjects.add(myGameObject);
myGameObject.body.setUserValue(gameObjects.size-1);
```

追加データを付けたい場合は`userData`フィールドが使えます。例は次のようなものです。
```java
btCollisionObject obj = new btCollisionObject();
obj.userData = myGameObject;
...
if (obj.userData instanceof MyGameObject)
  myGameObject = (MyGameObject)obj.userData;
```

`btCollisionObject`には`takeOwnership`と`releaseOwnership`というメソッドも追加されています。これらは、Javaオブジェクトがガベージコレクタによって破棄される際に、ネイティブオブジェクトの破棄をラッパーが担当する／しないを切り替えるために使えます。

また、`btCollisionObject`には次のメソッドも追加されています。
 * `getAnisotropicFriction(Vector3)`
 * `getWorldTransform(Matrix4)`
 * `getInterpolationWorldTransform(Matrix4)`
 * `getInterpolationLinearVelocity(Vector3)`
 * `getInterpolationAngularVelocity(Vector3)`
 * `getContactCallbackFlag()`と`setContactCallbackFlag(int)`
 * `getContactCallbackFilter()`と`setContactCallbackFilter(int)`

### ClosestNotMeConvexResultCallback
`ClosestNotMeConvexResultCallback`クラスは、`ClosestConvexResultCallback`のカスタム実装です。指定したオブジェクト「以外」すべてを対象に`convexSweepTest`を行うために使えます。

### ClosestNotMeRayResultCallback

`ClosestNotMeRayResultCallback`クラスは、`ClosestRayResultCallback`のカスタム実装です。指定したオブジェクト「以外」すべてを対象に`rayTest`を行うために使えます。

### InternalTickCallback

`InternalTickCallback`は、`btDynamicsWorld#setInternalTickCallback`が要求するコールバックをJavaクラスに橋渡しするために実装されています。このクラスを拡張して`onInternalTick`メソッドをオーバーライドできます。`attach`と`detach`メソッドでティックコールバックの受信を開始／停止できます。

### btDefaultMotionState

場合によっては、`btMotionState`を拡張するより`btDefaultMotionState`を使ったほうが簡単です。`btDefaultMotionState`には次のカスタムメソッドが用意されています。
 * `getGraphicsWorldTrans(Matrix4)`
 * `getCenterOfMassOffset(Matrix4)`
 * `getStartWorldTrans(Matrix4)`
ただし、自分の実装で`btMotionState`を拡張する方法が推奨されます。

### btCompoundShape

`btCompoundShape`クラスは、子シェイプ（shape）への参照を保持できるようにします（自分で参照を保持しなくてよくなります）。使うには、`addChildShape`の第3引数`managed`を`true`にして呼び出します。`btCompoundShape`が削除されると、`managed`指定された子シェイプも削除される点に注意してください。そのため、`managed`にするシェイプはその`btCompoundShape`専用にする必要があります。

### btIndexedMesh

`btIndexedMesh`クラスには次のコンストラクタが追加されています。
 * `btIndexedMesh(Mesh)`
また、次のメソッドが追加されています。
 * `setTriangleIndexBase(ShortBuffer)`
 * `setVertexBase(FloatBuffer)`
 * `set(Mesh)`
これらにより、`Mesh`インスタンスや頂点／インデックスバッファから `btIndexedMesh`を簡単に生成・設定できます。なお、バッファ自体はラッパーによって管理されないため、オブジェクトより長く生存させる必要があります。

### btTriangleIndexVertexArray

`btTriangleIndexVertexArray`クラスは、内部に保持しているJava側の`btIndexedMesh`クラスへの参照を維持できるようにします。使うには、`addIndexedMesh`の最後の引数`managed`をtrueにして呼び出します。`btTriangleIndexVertexArray`が破棄されると、`managed`指定された`btIndexedMesh`の子も破棄されます。

また`btTriangleIndexVertexArray`クラスには、簡単に生成・設定できるように`addMesh`と`addModel`メソッド、および同様のコンストラクタが追加されています。

### btBvhTriangleMeshShape

`btBvhTriangleMeshShape`クラスは、Java側の`btStridingMeshInterface`クラスへの参照を維持できるようにします。使うには、コンストラクタ引数`managed`を`true`にして生成します。`btBvhTriangleMeshShape`が破棄されると、`managed`指定された`btStridingMeshInterface`も破棄されます。

また`btBvhTriangleMeshShape`クラスには、1つ以上の`Mesh`または`Model`インスタンスから簡単に生成できるコンストラクタも追加されています。

### btConvexHullShape

`btConvexHullShape`クラスには、便利なコンストラクタ`btConvexHullShape(btShapeHull)`が追加されています。

### btBroadphasePairArray

`btBroadphasePairArray`クラスには、内部の衝突オブジェクトをまとめて取得するためのメソッドが追加されています。
```java
btBroadphasePairArray.getCollisionObjects(Array<btCollisionObject> out, btCollisionObject other, int[] tempArray)
btBroadphasePairArray.getCollisionObjectsValue(int[] out, btCollisionObject other)
```
### FilterableVehicleRaycaster

`FilterableVehicleRaycaster`クラスは`btDefaultVehicleRaycaster`を拡張し、`group`と`mask`を使った衝突フィルタリングをサポートします。
```java
FilterableVehicleRaycaster raycaster = new FilterableVehicleRaycaster(dynamicsWorld);
raycaster.setCollisionFilterGroup(FILTER_GROUP);
raycaster.setCollisionFilterMask(FILTER_MASK);
btRaycastVehicle vehicle = new btRaycastVehicle(vehicleTuning, chassis, raycaster);
```
