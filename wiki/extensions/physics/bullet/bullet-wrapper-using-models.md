---
title: Bulletラッパーでモデルを使用する
---
## モデルを使用する
[モデル（`Model`）](/wiki/graphics/3d/models)と`ModelInstance`は、通常オブジェクトの見た目（描画）を表すために使われます。一方、これらのオブジェクトの物理（衝突・剛体）を表すためには`btCollisionObject`や`btRigidBody`を使います。

### モーションステートを使う
`ModelInstance`と`btRigidBody`の位置・向きを同期するために、Bulletには拡張可能な`btMotionState`クラスが用意されています。同期の基本例は次のとおりです。
```java
static class MyMotionState extends btMotionState {
    Matrix4 transform;
    @Override
    public void getWorldTransform (Matrix4 worldTrans) {
        worldTrans.set(transform);
    }
    @Override
    public void setWorldTransform (Matrix4 worldTrans) {
        transform.set(worldTrans);
    }
}
```
使い方は次のとおりです。
```java
btRigidBody body;
ModelInstance instance;
MyMotionState motionState;
...
motionState = new MyMotionState();
motionState.transform = instance.transform;
body.setMotionState(motionState);
```
これで、`btRigidBody`が動くたびに`ModelInstance`の位置と向きが（Bulletによって）更新されます。この方法は`ModelInstance`に限らず、`Renderable`のように`Matrix4`の変換（transform）を持つオブジェクトなら同様に機能します。さらに、モーションステートに簡単なロジックを追加することもできます。
```java
static class PlayerMotionState extends btMotionState {
    final static Vector3 position = new Vector3();
    Player player;
    @Override
    public void getWorldTransform (Matrix4 worldTrans) {
        worldTrans.set(player.transform);
    }
    @Override
    public void setWorldTransform (Matrix4 worldTrans) {
        player.transform.set(worldTrans);
        player.transform.getTranslation(position);
        if (position.y < 0)
            player.die();
    }
}
```
**注意：** `btRigidBody`の変換（位置と回転）は、通常重心（多くの場合シェイプの中心）を基準にしています。必要であれば、`btCompoundShape`を使って重心位置をずらすことができます。見た目のモデルの原点は、物理オブジェクトの原点と同じにしておくことが推奨されます。もしそれが難しい場合は、モーションステート側で変換を適切に調整してください。

> Bulletの変換がサポートするのは、平行移動（位置）と回転（向き）のみです。スケーリングなど、それ以外の変換はサポートされません。

モーションステートは不要になったら破棄する必要があります：`motionState.dispose();`。

### モデルから衝突オブジェクトを作る
モデルは、いくつかのプロパティを持つ三角形の集合を、特定の変換で描画するものです。モデルは描画向けに最適化されており、物理用途向けではありません。そのため、モデルがそのまま物理シェイプを効率よく表現できるケースは多くありません。

理由を理解するために、単純な箱のモデルを考えてみます。箱の物理シェイプは8つの角（頂点）で表せます。しかし、見た目のモデルは24個の角（頂点）を持ちます。これは、箱の各面ごとに頂点が定義され、各頂点がその面の「法線（normal）」を持つからです。そうしないと、ライティングなどの視覚効果が実現できません。つまり、見た目のモデルは「中身が詰まった箱」ではなく、実際には6枚の独立した長方形（またはそれを構成する三角形）からできています。これらの長方形（や三角形）は無限に薄く、体積がありません。そのため、動的な物理には不向きです。

ほかにも問題はあります。たとえばモデルは、物理に必要な以上に細かいディテールを持っていることが一般的です。実際、形状によってはモデルの頂点を使うよりも、ずっと低コストな衝突判定アルゴリズムが使えます。箱形状であれば、12枚の三角形に対する判定ではなく、単一の「箱」に対する判定で済ませられます。

これらの問題を回避する方法はいくつかあります。プリミティブ形状でモデルを近似する方法から、専用のモデルを用意する方法、見た目モデルと物理シェイプの間で頂点を共有する方法までさまざまです。[Bulletマニュアル](https://github.com/bulletphysics/bullet3/blob/master/docs/Bullet_User_Manual.pdf?raw=true)には、どの方法を選ぶべきか判断するためのチャートが用意されています。
  
![images/bullet_shape_decision.png](/assets/wiki/images/bullet_shape_decision.png)

静的モデルの場合、Bulletラッパーにはモデルから衝突シェイプを作るための便利なメソッドがあります。
```java
btCollisionShape shape = Bullet.obtainStaticNodeShape(model.nodes);
```
この場合、衝突シェイプはモデルと同じデータ（頂点）を共有します。必要であれば`btCompoundShape`を使って[ノード変換](/wiki/graphics/3d/models#node-transformation)も取り込みますが、ノードに適用されたスケーリングは取り込みません。
