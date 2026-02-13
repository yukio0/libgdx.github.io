---
title: Box2d
---
# libGDXでBox2Dをセットアップする

Box2Dは2D物理演算ライブラリです。2Dゲーム向け物理ライブラリの中でも特に人気が高く、libGDXを含む多くの言語・エンジンへ移植されています。libGDXにおけるBox2Dの実装は、C++エンジンを最小限にラップしたJavaラッパーです。そのため、[Box2Dドキュメント](https://box2d.org/documentation/)が役に立つことがあります。

Box2Dは拡張機能であり、libGDXにデフォルトでは含まれていません。したがって手動での導入が必要です。

## 目次

  * [初期化](/wiki/extensions/physics/box2d#initialization)
  * [Worldの作成](/wiki/extensions/physics/box2d#creating-a-world)
  * [デバッグレンダラー](/wiki/extensions/physics/box2d#debug-renderer)
  * [シミュレーションを進める](/wiki/extensions/physics/box2d#stepping-the-simulation)
  * [描画（レンダリング）](/wiki/extensions/physics/box2d#rendering)
  * [オブジェクト／ボディ](/wiki/extensions/physics/box2d#objectsbodies)
    * [動的ボディ](/wiki/extensions/physics/box2d#dynamic-bodies)
    * [静的ボディ](/wiki/extensions/physics/box2d#static-bodies)
    * [キネマティックボディ](/wiki/extensions/physics/box2d#kinematic-bodies)
  * [インパルス／力](/wiki/extensions/physics/box2d#impulsesforces)
  * [ジョイントとギア](/wiki/extensions/physics/box2d#joints-and-gears)
  * [フィクスチャ形状](/wiki/extensions/physics/box2d#fixture-shapes)
  * [スプライトとボディ](/wiki/extensions/physics/box2d#sprites-and-bodies)
  * [センサー](/wiki/extensions/physics/box2d#sensors)
  * [衝突リスナー](/wiki/extensions/physics/box2d#contact-listeners)
  * [参考資料](/wiki/extensions/physics/box2d#resources)
  * [ツール](/wiki/extensions/physics/box2d#tools)

## 初期化 {#initialization}

Box2Dを初期化するには`Box2D.init()`を呼ぶ必要があります。後方互換性のため、最初に`World`を作成したときにも同様の効果がありますが、`Box2D`クラスを使う方法を推奨します。

## Worldの作成  {#creating-a-world}

Box2Dをセットアップする際、最初に必要になるのは`World`です。`World`オブジェクトは、物理オブジェクト／ボディをすべて保持し、それらの間の反応をシミュレートするものです。ただし、オブジェクトの描画は行いません。描画にはlibGDXのグラフィックス機能を使います。とはいえ、libGDXにはBox2Dのデバッグレンダラーが用意されており、物理シミュレーションのデバッグや、レンダリングコードを書く前のゲームプレイ検証にも非常に便利です。

Worldは次のコードで作成します。

```java
World world = new World(new Vector2(0, -10), true);
```

第1引数は重力を表す2Dベクトルです。0は水平（x方向）の重力、-10は現実世界のような下向きの力（y軸が上向きの場合）を意味します。値は自由に決められますが、スケールは一定に保ってください。Box2Dでは1単位=1メートルです。

`World`作成時の第2引数は`boolean`で、オブジェクトをスリープさせるかどうかを指定します。通常はCPU使用量を抑えるためスリープを有効にしますが、状況によってはスリープさせたくないこともあります。

Box2Dで使うスケールと、グラフィックス描画のスケールを揃えることが推奨されます。つまり、スプライト（sprite）の幅／高さもメートル単位で描くということです。見える大きさにするには、viewportWidth/viewportHeightもメートル単位のカメラを使って拡大します。例として、幅2.0f（2m）のスプライトを描き、カメラのviewportWidthを20.0fにすると、そのスプライトはウィンドウ幅の1/10を占めます。

**よくあるミス**は、メートルではなくピクセルで世界を測ってしまうことです。Box2Dのオブジェクトは移動速度に限界があります。メートル（例：12×9）の代わりにピクセル（例：640×480）を使ってしまうと、何をしても常に遅く動くように見えてしまいます。

## デバッグレンダラー {#debug-renderer}

次にデバッグレンダラーをセットアップします。リリース版では通常使いませんが、テスト目的で次のように作成します。

```java
Box2DDebugRenderer debugRenderer = new Box2DDebugRenderer();
```



## シミュレーションを進める {#stepping-the-simulation}

シミュレーションを更新するには、`World`に`step`を呼び出します。`step`は、時間経過に沿って`World`内のオブジェクトを更新します。呼び出す場所としては、`render()`ループの最後が最適です。理想的な世界では、全員のフレームレートが同じですが……

```java
world.step(1/60f, 6, 2);
```

第1引数はタイムステップ、つまり`World`にシミュレートさせたい時間量です。多くの場合、固定タイムステップにします。libGDXは`1/60f`（1/60 秒）〜`1/240f`（1/240 秒）の値を推奨しています。

残り2つは`velocityIterations`と`positionIterations`です。ここではひとまず`6`と`2`のままにしますが、詳細はBox2Dドキュメントを参照してください。

シミュレーションのステップは、それ自体が大きなテーマです。可変タイムステップの扱いについては、優れた解説として[この記事](https://gafferongames.com/post/fix_your_timestep/)を参照してください。

例えば、次のような実装になります。

```java
private float accumulator = 0;

private void doPhysicsStep(float deltaTime) {
    // 固定タイムステップ
    // 遅い端末で、更新が追いつかず悪化する「スパイラル・オブ・デス」を防ぐため、フレーム時間の上限を設ける
    float frameTime = Math.min(deltaTime, 0.25f);
    accumulator += frameTime;
    while (accumulator >= Constants.TIME_STEP) {
        WorldManager.world.step(Constants.TIME_STEP, Constants.VELOCITY_ITERATIONS, Constants.POSITION_ITERATIONS);
        accumulator -= Constants.TIME_STEP;
    }
}
```

## 描画（レンダリング） {#rendering}

物理ステップよりも先にすべてのグラフィックスを描画することが推奨されます。そうしないと同期がズレます。デバッグレンダラーで描画する場合は次のとおりです。

```java
debugRenderer.render(world, camera.combined);
```

第1引数はBox2DのWorld、第2引数はlibGDXのカメラです。



## オブジェクト／ボディ {#objectsbodies}

このままゲームを実行しても、何も起きないので退屈です。Worldはステップしているのに、相互作用する対象がないからです。そこでオブジェクトを追加していきます。

Box2Dではオブジェクトを*ボディ*（body）と呼びます。各ボディは1つ以上の*フィクスチャ*（fixture）から成り、フィクスチャはボディの中で固定の位置と向きを持ちます。フィクスチャの形状は自由で、複数の異なる形状を組み合わせて望む形にすることもできます。

フィクスチャには形状（shape）、密度（density）、摩擦（friction）、反発係数（restitution）が紐づきます。形状はそのまま。密度は平方メートルあたりの質量のことで、ボウリング球は密度が高い一方、風船はほとんど空気なので密度が低いです。摩擦は、物体が何かに擦れたり滑ったりする際の抵抗のことで、氷のブロックは摩擦が低く、ゴムボールは高いでしょう。反発係数は弾み具合のことで、岩は低く、バスケットボールは比較的高いです。反発係数0のボディは地面に当たるとすぐ止まり、反発係数1のボディは永遠に同じ高さまで跳ね続けます。

ボディには3種類あります：動的（dynamic）、キネマティック（kinematic）、静的（static）です。それぞれ以下で説明します。


### 動的ボディ {#dynamic-bodies}

動的ボディ（dynamic body）は、動き回り、力の影響を受けるオブジェクトです。また、他の動的／キネマティック／静的オブジェクトの影響も受けます。力を受けて動く必要があるオブジェクトには、動的ボディが適しています。

ここまでで、ボディを構成するフィクスチャについて学びました。では実際にボディを作り、フィクスチャを付けていきましょう。

```java
// まずボディ定義を作成する
BodyDef bodyDef = new BodyDef();
// ボディを動的にする。地面のように動かないものは静的ボディにする
bodyDef.type = BodyType.DynamicBody;
// World内での初期位置を設定する
bodyDef.position.set(5, 10);

// ボディ定義を使ってWorldにボディを作成する
Body body = world.createBody(bodyDef);

// 円形シェイプを作成し、半径を6にする
CircleShape circle = new CircleShape();
circle.setRadius(6f);

// シェイプを適用するためのフィクスチャ定義を作成する
FixtureDef fixtureDef = new FixtureDef();
fixtureDef.shape = circle;
fixtureDef.density = 0.5f;
fixtureDef.friction = 0.4f;
fixtureDef.restitution = 0.6f; // 少し弾むようにする

// フィクスチャを作成し、ボディに取り付ける
Fixture fixture = body.createFixture(fixtureDef);

// 使い終わったシェイプは必ずdisposeすること！
// BodyDefとFixtureDefはdispose不要だが、シェイプは必要。
circle.dispose();
```

これで、ボールのようなオブジェクトを作ってWorldに追加できました。ゲームを実行すると、ボールが画面の下へ落ちていくのが見えるはずです。とはいえ、まだ相互作用する相手がいないので退屈ですね。次は、ボールが跳ねる床を作りましょう。



### 静的ボディ {#static-bodies}

静的ボディ（static body）は、動かず、力の影響も受けないオブジェクトです。動的ボディは静的ボディの影響を受けます。静的ボディは地面や壁など、動く必要のないものに最適です。また、必要な計算量も少なくて済みます。

では、床を静的ボディとして作成してみましょう。手順は、先ほど動的ボディを作ったときとよく似ています。

```java
// ボディ定義を作成する
BodyDef groundBodyDef = new BodyDef();  
// World内での位置を設定する
groundBodyDef.position.set(new Vector2(0, 10));  

// 定義からボディを作成してWorldに追加する
Body groundBody = world.createBody(groundBodyDef);  

// ポリゴンシェイプを作成する
PolygonShape groundBox = new PolygonShape();  
// ビューポート幅の2倍で、高さ20の箱として設定する
// （setAsBoxは、半分の幅と半分の高さを引数に取る）
groundBox.setAsBox(camera.viewportWidth, 10.0f);
// ポリゴンシェイプからフィクスチャを作成し、床ボディに追加する
groundBody.createFixture(groundBox, 0.0f);
// 後片付け
groundBox.dispose();
```

`FixtureDef`を定義しなくてもフィクスチャを作れたのが分かるでしょうか。指定したいのが「シェイプ」と「密度」だけであれば、`createFixture`には便利なオーバーロードが用意されています。

この状態でゲームを実行すると、ボールが落下し、新しく作った地面で跳ねるのが見えるはずです。密度や反発係数などの値をいろいろ変えて、挙動がどう変わるか試してみてください。



### キネマティックボディ {#kinematic-bodies}

キネマティックボディ（kinematic body）は、静的ボディと動的ボディの中間のような存在です。静的ボディと同様に力には反応しませんが、動的ボディのように動くことはできます。プラットフォームゲームの動く足場のように、「プログラマがボディの動きを完全に制御したい」ものに向いています。

キネマティックボディは位置を直接設定することもできますが、一般的には速度を設定し、位置更新はBox2Dに任せるほうが良いでしょう。

キネマティックボディも、作り方は動的／静的ボディとほぼ同じです。作成後は、例えば次のように速度を制御できます。

```java
// 1秒あたり1メートルで上方向へ移動する
kinematicBody.setLinearVelocity(0.0f, 1.0f);
```

## インパルス／力 {#impulsesforces}

インパルス（impulse）と力（force）は、重力や衝突に加えてボディを動かすために使います。

力は時間をかけて徐々に作用し、ボディの速度を変化させます。例えば、ロケットの離陸では、徐々に力が加わっていき、ゆっくり加速していきます。

一方インパルスは、ボディの速度を即座に変化させます。例えば、パックマンのようにキャラクターが一定速度で移動し、入力と同時に速度が決まるような動きは、インパルスのイメージに近いでしょう。

インパルス／力を適用するには動的ボディが必要です。上の[動的ボディ](#dynamic-bodies)を参照してください。

**力を加える**

力は、World座標上の一点に対してニュートン（N）で加えます。力を重心に加えない場合、トルクが発生して角速度にも影響します。

```java
// ボディのpos.x／pos.yに、X軸方向へ1（N）の力を加えて、ゆっくり右へ動かす
dynamicBody.applyForce(1.0f, 0.0f, pos.x, pos.y, true);

// 常にボディ中心に力を加える場合
dynamicBody.applyForceToCenter(1.0f, 0.0f, true);
```

**インパルスを加える**

インパルスは、力と似ていますが「即座に」速度を変える点が異なります。力と同様、重心以外にインパルスを加えるとトルクが発生し、角速度が変化します。インパルスの単位はニュートン秒（N·s）またはkg·m/sです。

```java
// X方向の速度を即座に1m/sにして、素早く右へ動かす
dynamicBody.applyLinearImpulse(1.0f, 0, pos.x, pos.y, true);
```

また、力やインパルスを加えるとボディはスリープ解除されます。場合によっては望ましくないこともあります。例えば、一定の力を加え続けつつ、パフォーマンスのためにボディがスリープできるようにしたい場合などです。その場合は、`wake`引数を`false`にできます。

```java
// インパルスを加えるが、スリープ解除しない
dynamicBody.applyLinearImpulse(0.8f, 0, pos.x, pos.y, false);
```

**プレイヤー移動の例**

この例では、ソニックのように、プレイヤーが左右に走って加速し、最大速度に達したらそれ以上加速しない動きを作ります。ここでは、すでに`player`という名前の動的ボディがあり、最大速度を示す`MAX_VELOCITY`変数も定義済みとします。あとは、キー入力時に線形インパルスを加えるだけです。

```java
Vector2 vel = this.player.body.getLinearVelocity();
Vector2 pos = this.player.body.getPosition();

// 左へのインパルス（まだ最大速度に達していない場合のみ）
if (Gdx.input.isKeyPressed(Keys.A) && vel.x > -MAX_VELOCITY) {			
     this.player.body.applyLinearImpulse(-0.80f, 0, pos.x, pos.y, true);
}

// 右へのインパルス（まだ最大速度に達していない場合のみ）
if (Gdx.input.isKeyPressed(Keys.D) && vel.x < MAX_VELOCITY) {
     this.player.body.applyLinearImpulse(0.80f, 0, pos.x, pos.y, true);
}
```

## ジョイントとギア {#joints-and-gears}

どのジョイント（joint）も、Box2DのWorldで作成する前に定義を設定する必要があります。`initialize`を使うと、ジョイントに必要なパラメータが一通り設定されるので便利です。

また、ボディを破棄した後にそのジョイントを破棄するとクラッシュの原因になります。ボディを破棄すると、そのボディに接続されているジョイントも一緒に破棄されるためです。

```java
DistanceJointDef defJoint = new DistanceJointDef ();
defJoint.length = 0;
defJoint.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));

DistanceJoint joint = (DistanceJoint) world.createJoint(defJoints); // Jointのサブクラスが返る
```

### 距離ジョイント

距離ジョイント（distance joint）は、2つのボディ間の距離を一定に保ちます。

距離ジョイントの定義では、両方のボディにアンカーポイントを設定し、距離ジョイントの長さ（0ではない値）を指定する必要があります。

この定義はローカル座標のアンカーポイントを使うため、初期状態が制約に対してわずかに矛盾していても構いません。これはゲームのセーブ／ロード時に役立ちます。

**長さを0や極端に短い値にしないでください！**

```java
// DistanceJointDef.initialize (Body bodyA, Body bodyB, Vector2 anchorA, Vector2 anchorB)

DistanceJointDef defJoint = new DistanceJointDef ();
defJoint.length = 0;
defJoint.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));
```

### 摩擦ジョイント

摩擦ジョイント（friction joint）は、トップダウン（見下ろし）視点の摩擦表現などに使えます。2Dの並進摩擦と角度方向の摩擦を提供します。

```java
FrictionJointDef jointDef = new FrictionJointDef ();
jointDef.maxForce = 1f;
jointDef.maxTorque = 1f;
jointDef.initialize(bodyA, bodyB, anchor);
```

### ギアジョイント

ギアジョイント（gear joint）は、2つのジョイントを連結するために使います。連結対象は回転（revolute）ジョイントでも並進（prismatic）ジョイントでも構いません。ギア比（ratio）を指定して、次の関係で動きを束ねます。

coordinate1 + ratio * coordinate2 = constant

ratioは正にも負にもできます。片方が回転ジョイントで、もう片方が並進ジョイントの場合、ratioは「長さ」または「1/長さ」の単位を持ちます。

```java
GearJointDef jointDef = new GearJointDef (); // initializeはない
```

### モータージョイント
モータージョイント（motor joint）は、2つのボディの相対運動を制御するために使います。典型的な用途は、地面に対して動的ボディの動きを制御することです。

```java
MotorJointDef jointDef = new MotorJointDef ();
jointDef.angularOffset = 0f;
jointDef.collideConnected = false;
jointDef.correctionFactor = 0f;
jointDef.maxForce = 1f;
jointDef.maxTorque = 1f;
jointDef.initialize(bodyA, bodyB);
```

### マウスジョイント
マウスジョイント（mouse joint）はテストベッドで、マウス操作によりボディを動かすために使われます。ボディ上の一点を、カーソルの現在位置に向かって引っ張るように動かします。回転には制限がありません。

```java
MouseJointDef jointDef = new MouseJointDef();
jointDef.target = new Vector2(Gdx.input.getX(), Gdx.input.getY());

MouseJoint joint = (MouseJoint) world.createJoint(jointDef);
joint.setTarget(new Vector2(Gdx.input.getX(), Gdx.input.getY()));
```

### 並進ジョイント

並進ジョイント（prismatic joint）は、指定した軸に沿って2つのボディが相対的に平行移動できるようにします。一方で相対回転は防ぎます。つまり自由度は1つです。

```java
PrismaticJointDef jointDef = new PrismaticJointDef ();
jointDef.lowerTranslation = -5.0f;
jointDef.upperTranslation = 2.5f;

jointDef.enableLimit = true;
jointDef.enableMotor = true;

jointDef.maxMotorForce = 1.0f;
jointDef.motorSpeed = 0.0f;
```

### 滑車ジョイント

滑車ジョイント（pulley joint）は、理想化した滑車を作るために使います。滑車は2つのボディを地面および互いに接続します。片方のボディが上がると、もう片方が下がります。滑車のロープ全長は、初期構成に基づいて保存されます。

```java
JointDef jointDef = new JointDef ();
float ratio = 1.0f;
jointDef.Initialize(myBody1, myBody2, groundAnchor1, groundAnchor2, anchor1, anchor2, ratio);
```

### 回転ジョイント

回転ジョイント（revolute joint）は、2つのボディが共通のアンカーポイント（ヒンジ点）を共有するように強制します。回転ジョイントの自由度は1つで、2つのボディの相対回転です。これをジョイント角と呼びます。

```java
RevoluteJointDef jointDef = new RevoluteJoint();
jointDef.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));

jointDef.lowerAngle = -0.5f * b2_pi; // -90度
jointDef.upperAngle = 0.25f * b2_pi; // 45度

jointDef.enableLimit = true;
jointDef.enableMotor = true;

jointDef.maxMotorTorque = 10.0f;
jointDef.motorSpeed = 0.0f;
```

### ロープジョイント

ロープジョイント（rope joint）は、2つのボディ上の2点の最大距離を制限します。それ以外の効果はありません。注意：シミュレーション中に最大長を変更しようとすると、物理的でない挙動が発生します。長さを動的に変更できるモデルには「伸び（スポンジ感）」が必要になるため、この実装ではそうしないようにしています。長さを動的に制御したい場合は`b2DistanceJoint`を参照してください。

```java
RopeJointDef jointDef = new RopeJointDef (); // initializeはない
```

### 溶接ジョイント

溶接ジョイント（weld joint）は、2つのボディを「接着」するようなジョイントです。アイランド制約ソルバが近似的であるため、溶接ジョイントは多少歪むことがあります。

```java
WeldJointDef jointDef = new WeldJointDef ();
jointDef.initialize(bodyA, bodyB, anchor);
```

### ホイールジョイント

ホイールジョイント（wheel joint）は、2つの自由度を提供します。ひとつはbodyAに固定された軸に沿った並進、もうひとつは平面内の回転です。ジョイントのリミットで可動範囲を制限でき、モーターで回転を駆動したり、回転摩擦をモデル化したりできます。車両のサスペンション向けに設計されています。

```java
WheelJointDef jointDef = new WheelJointDef();
jointDef.maxMotorTorque = 1f;
jointDef.motorSpeed = 0f;
jointDef.dampingRatio = 1f;
jointDef.initialize(bodyA, bodyB, anchor, axis); // axisはVector2(1,1)

WheelJoint joint = (WheelJoint) physics.createJoint(jointDef);
joint.setMotorSpeed(1f);
```

## フィクスチャ形状 {#fixture-shapes}

前述のとおり、フィクスチャには形状（shape）、密度（density）、摩擦（friction）、反発係数（restitution）が紐づきます。
Box2Dでは、標準機能だけでも箱（[静的ボディ](/wiki/extensions/physics/box2d#static-bodies)のセクション参照）や円形（[動的ボディ](/wiki/extensions/physics/box2d#dynamic-bodies)のセクション参照）の形状を簡単に作れます。

より複雑な形状は、次のクラスを使ってプログラムから定義できます。
* ChainShape,
* EdgeShape,
* PolygonShape

ただし、サードパーティ製ツールを使えば、形状をツール側で作ってゲームへ取り込むだけ、という運用もできます。

### box2d-editorを使った複雑形状の取り込み

[box2d-editor](https://github.com/julienvillegas/box2d-editor)は、複雑な形状を定義し、ゲームへ読み込むための無料・オープンソースのツールです。
box2d-editorで作った形状をゲームへ取り込む例は、[Libgdx.info](https://libgdxinfo.wordpress.com/box2d-importing-complex-bodies/)にあります。

他のツールも知りたい場合は、[ツールのセクション](/wiki/extensions/physics/box2d#Tools)も参照してください。

要するに、Box2d-editorを使う場合の流れはこうです。
* Box2d-editorで形状を作成する
* シーンをエクスポートし、生成されたファイルをassetsフォルダへコピーする
* BodyEditorLoader.javaを「core」モジュールのソースフォルダへコピーする

するとゲーム側では、次のように書けます。

```java
BodyEditorLoader loader = new BodyEditorLoader(Gdx.files.internal("box2d_scene.json"));

BodyDef bd = new BodyDef();
bd.type = BodyDef.BodyType.KinematicBody;
body = world.createBody(bd);

// 2. いつも通りFixtureDefを作成する
FixtureDef fd = new FixtureDef();
fd.density = 1;
fd.friction = 0.5f;
fd.restitution = 0.3f;

// 3. いつも通りボディに取り付ける
loader.attachFixture(body, "gear", fd, scale);
```

## スプライトとボディ {#sprites-and-bodies}

スプライト／ゲームオブジェクトとBox2Dを結び付ける最も簡単な方法は、Box2Dのユーザーデータを使うことです。ゲームオブジェクトをユーザーデータとして設定しておき、Box2Dボディの位置に合わせてオブジェクト側を更新します。

ボディにユーザーデータを設定するのは簡単です。

```java
body.setUserData(Object);
```

ここには任意のJavaオブジェクトを設定できます。物理ボディへの参照を保持できるよう、自分用のActor／Objectクラスを用意しておくのも良いでしょう。

フィクスチャにも同様にユーザーデータを設定できます。

```java
fixture.setUserData(Object);
```

すべてのActor／Spriteを更新するには、ゲーム（またはrender）ループの中でWorld内のボディを走査するのが手軽です。

```java
// ボディを格納する配列を用意する
// （毎回newしないほうが良い）
Array<Body> bodies = new Array<Body>();
// World内のボディを配列に詰める
world.getBodies(bodies);

for (Body b : bodies) {
    // ボディのユーザーデータを取得する（例：Entityクラスのインスタンス）
    Entity e = (Entity) b.getUserData();

    if (e != null) {
        // エンティティ／スプライトの位置と角度を更新する
        e.setPosition(b.getPosition().x, b.getPosition().y);
        // 角度はラジアン→度へ変換が必要
        e.setRotation(MathUtils.radiansToDegrees * b.getAngle());
    }
}
```

あとは通常どおり、libGDXの`SpriteBatch`でスプライトを描画してください。

## センサー {#sensors}
センサー（sensor）は、衝突が起きても自動的な物理反応（力を加える等）を発生させないボディです。2つの形状が衝突したときの挙動を「完全に自分で制御したい」場合に便利です。
例えば、ドローンが円形の索敵範囲を持っているとします。この範囲はドローンに追従する必要がありますが、ドローン自身や他のボディと物理的に反応してほしくはありません。必要なのは「ターゲットが範囲内に入ったかどうか」の検知だけです。

ボディをセンサーにするには、`isSensor`フラグを`true`にします。例えば次のように設定します。

```java
// Fixture定義時に設定する
fixtureDef.isSensor = true;
```

このセンサー接触を監視するには、`ContactListener`インターフェースのメソッドを実装する必要があります。

## 衝突リスナー {#contact-listeners}
衝突リスナー（contact listener）は、特定のフィクスチャで発生した衝突イベントを監視します。各メソッドには`Contact`オブジェクトが渡され、そこには衝突に関わった2つのボディに関する情報が入っています。
`beginContact`はオブジェクト同士が重なったときに呼ばれ、衝突しなくなったときには`endContact`が呼ばれます。

```java
public class ListenerClass implements ContactListener {
		@Override
		public void endContact(Contact contact) {

		}

		@Override
		public void beginContact(Contact contact) {

		}
	};
```

このクラスは、screenの`show()`または`init()`メソッドでWorldの衝突リスナーとして設定する必要があります。

```java
world.setContactListener(ListenerClass);
```

接触したフィクスチャからボディ情報を取得できる場合があります。
アプリケーション設計によっては、Entityクラスの参照をボディやフィクスチャのユーザーデータに入れておき、Contactから取り出して必要な処理（例：プレイヤーの体力を減らす）を行えるようにしておくと良いでしょう。

## 参考資料 {#resources}

Box2Dには優れた参考資料が多数あり、その多くのコードはlibGDX用にも比較的簡単に移植できます。

  * Scene2DとBox2Dを組み合わせた基本実装とコード例は、[LibGDX.info](https://libgdxinfo.wordpress.com/box2d-basic/)にもあります。
  * [Box2Dドキュメント](https://box2d.org/documentation/)と[Discord](https://discord.com/invite/NKYgCBP)は、困ったときの助けになります。
  * とても良い[Box2Dチュートリアル連載](https://www.iforce2d.net/b2dtut/)もあります。ゲーム開発で遭遇しがちな問題を幅広く扱っています。

## ツール {#tools}

box2dとlibGDXで使えるツール一覧です。

### 無料・オープンソース

  * [Physics Body Editor](https://github.com/julienvillegas/box2d-editor)

コードサンプル： [https://libgdxinfo.wordpress.com](https://libgdxinfo.wordpress.com/box2d-importing-complex-bodies//)

### 商用

  * [RUBE](https://www.iforce2d.net/rube/)：box2dのワールド作成用エディタ。RUBEデータをlibGDXに読み込むには[RubeLoader](https://github.com/indiumindeed/RubeLoader)を使用します。
  * [PhysicsEditor](https://www.codeandweb.com/physicseditor)
