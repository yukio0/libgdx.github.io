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
  * [描画](/wiki/extensions/physics/box2d#rendering)
  * [オブジェクト／ボディ](/wiki/extensions/physics/box2d#objectsbodies)
    * [動的ボディ](/wiki/extensions/physics/box2d#dynamic-bodies)
    * [静的ボディ](/wiki/extensions/physics/box2d#static-bodies)
    * [キネマティックボディ](/wiki/extensions/physics/box2d#kinematic-bodies)
  * [Impulses/Forces](/wiki/extensions/physics/box2d#impulsesforces)
  * [Joints and Gears](/wiki/extensions/physics/box2d#joints-and-gears)
  * [Fixture Shapes](/wiki/extensions/physics/box2d#fixture-shapes)
  * [Sprites and Bodies](/wiki/extensions/physics/box2d#sprites-and-bodies)
  * [Sensors](/wiki/extensions/physics/box2d#sensors)
  * [Contact Listeners](/wiki/extensions/physics/box2d#contact-listeners)
  * [Resources](/wiki/extensions/physics/box2d#resources)
  * [Tools](/wiki/extensions/physics/box2d#tools)

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

## 描画 {#rendering}

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

キネマティックボディ（kinematic bosy）は、静的ボディと動的ボディの中間のような存在です。静的ボディと同様に力には反応しませんが、動的ボディのように動くことはできます。プラットフォームゲームの動く足場のように、「プログラマがボディの動きを完全に制御したい」ものに向いています。

キネマティックボディは位置を直接設定することもできますが、一般的には速度を設定し、位置更新はBox2Dに任せるほうが良いでしょう。

キネマティックボディも、作り方は動的／静的ボディとほぼ同じです。作成後は、例えば次のように速度を制御できます。

```java
// 1秒あたり1メートルで上方向へ移動する
kinematicBody.setLinearVelocity(0.0f, 1.0f);
```

## Impulses/Forces

Impulses and Forces are used to move a body in addition to gravity and collision.

Forces occur gradually over time to change the velocity of a body. For example, a rocket lifting off would slowly have forces applied as the rocket slowly begins to accelerate.

Impulses on the other hand make immediate changes to the body's velocity. For example, playing Pac-Man the character always moved at a constant speed and achieved instant velocity upon being moved.

First you will need a Dynamic Body to apply forces/impulses to, see the [Dynamic Bodies](#dynamic_bodies) section above.

**Applying Force**

Forces are applied in Newtons at a World Point. If the force is not applied to the center of mass, it will generate torque and affect the angular velocity.

```java
// Apply a force of 1 meter per second on the X-axis at pos.x/pos.y of the body slowly moving it right
dynamicBody.applyForce(1.0f, 0.0f, pos.x, pos.y, true);

// If we always want to apply force at the center of the body, use the following
dynamicBody.applyForceToCenter(1.0f, 0.0f, true);
```

**Applying Impulse**

Impulses are just like Forces with the exception that they immediately modify the velocity of a body. As with forces, if the impulse is not applied at the center of a body, it will create torque which modifies angular velocity. Impulses are applied in Newton-seconds or kg-m/s.

```java
// Immediately set the X-velocity to 1 meter per second causing the body to move right quickly
dynamicBody.applyLinearImpulse(1.0f, 0, pos.x, pos.y, true);
```

Keep in mind applying forces or impulses will wake the body. Sometimes this behavior is undesired. For example, you may be applying a steady force and want to allow the body to sleep to improve performance. In this case you can set the wake boolean value to false.

```java
// Apply impulse but don't wake the body
dynamicBody.applyLinearImpulse(0.8f, 0, pos.x, pos.y, false);
```

**Player Movement Example**

In this example, we will make a player run left or right and accelerate to a maximum velocity, just like Sonic the Hedgehog. For this example we have already created a Dynamic Body named 'player'. In addition we have defined a MAX_VELOCITY variable so our player won't accelerate beyond this value. Now it's just a matter of applying a linear impulse when a key is pressed.

```java
Vector2 vel = this.player.body.getLinearVelocity();
Vector2 pos = this.player.body.getPosition();

// apply left impulse, but only if max velocity is not reached yet
if (Gdx.input.isKeyPressed(Keys.A) && vel.x > -MAX_VELOCITY) {			
     this.player.body.applyLinearImpulse(-0.80f, 0, pos.x, pos.y, true);
}

// apply right impulse, but only if max velocity is not reached yet
if (Gdx.input.isKeyPressed(Keys.D) && vel.x < MAX_VELOCITY) {
     this.player.body.applyLinearImpulse(0.80f, 0, pos.x, pos.y, true);
}
```

## Joints and Gears

Every joint requires to have definition set up before creating it by box2d world. Using *initialize* helps with ensuring that all joint parameters are set.

Note that destroying the joint after the body will cause crash. Destroying the body also destroys joints connected to it.

```java
DistanceJointDef defJoint = new DistanceJointDef ();
defJoint.length = 0;
defJoint.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));

DistanceJoint joint = (DistanceJoint) world.createJoint(defJoints); // Returns subclass Joint.
```

### DistanceJoint

Distance joint makes length between bodies constant.

Distance joint definition requires defining an anchor point on both bodies and the non-zero length of the distance joint.

The definition uses local anchor points so that the initial configuration can violate the constraint slightly. This helps when saving and loading a game.

**Do not use a zero or short length!**

```java
// DistanceJointDef.initialize (Body bodyA, Body bodyB, Vector2 anchorA, Vector2 anchorB)

DistanceJointDef defJoint = new DistanceJointDef ();
defJoint.length = 0;
defJoint.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));
```

### FrictionJoint

Friction joint is used for top-down friction. It provides 2D translational friction and angular friction.

```java
FrictionJointDef jointDef = new FrictionJointDef ();
jointDef.maxForce = 1f;
jointDef.maxTorque = 1f;
jointDef.initialize(bodyA, bodyB, anchor);
```

### GearJoint

A gear joint is used to connect two joints together. Either joint can be a revolute or prismatic joint. You specify a gear ratio to bind the motions together: coordinate1 + ratio * coordinate2 = constant The ratio can be negative or positive. If one joint is a revolute joint and the other joint is a prismatic joint, then the ratio will have units of length or units of 1/length.

```java
GearJointDef jointDef = new GearJointDef (); // has no initialize
```

### MotorJoint
A motor joint is used to control the relative motion between two bodies. A typical usage is to control the movement of a dynamic body with respect to the ground.

```java
MotorJointDef jointDef = new MotorJointDef ();
jointDef.angularOffset = 0f;
jointDef.collideConnected = false;
jointDef.correctionFactor = 0f;
jointDef.maxForce = 1f;
jointDef.maxTorque = 1f;
jointDef.initialize(bodyA, bodyB);
```

### MouseJoint
The mouse joint is used in the testbed to manipulate bodies with the mouse. It attempts to drive a point on a body towards the current position of the cursor. There is no restriction on rotation.

```java
MouseJointDef jointDef = new MouseJointDef();
jointDef.target = new Vector2(Gdx.input.getX(), Gdx.input.getY());

MouseJoint joint = (MouseJoint) world.createJoint(jointDef);
joint.setTarget(new Vector2(Gdx.input.getX(), Gdx.input.getY()));
```

### PrismaticJoint

A prismatic joint allows for relative translation of two bodies along a specified axis. A prismatic joint prevents relative rotation. Therefore, a prismatic joint has a single degree of freedom.

```java
PrismaticJointDef jointDef = new PrismaticJointDef ();
jointDef.lowerTranslation = -5.0f;
jointDef.upperTranslation = 2.5f;

jointDef.enableLimit = true;
jointDef.enableMotor = true;

jointDef.maxMotorForce = 1.0f;
jointDef.motorSpeed = 0.0f;
```

### PulleyJoint

A pulley is used to create an idealized pulley. The pulley connects two bodies to ground and to each other. As one body goes up, the other goes down. The total length of the pulley rope is conserved according to the initial configuration.

```java
JointDef jointDef = new JointDef ();
float ratio = 1.0f;
jointDef.Initialize(myBody1, myBody2, groundAnchor1, groundAnchor2, anchor1, anchor2, ratio);
```

### RevoluteJoint

A revolute joint forces two bodies to share a common anchor point, often called a hinge point. The revolute joint has a single degree of freedom: the relative rotation of the two bodies. This is called the joint angle

```java
RevoluteJointDef jointDef = new RevoluteJoint();
jointDef.initialize(bodyA, bodyB, new Vector2(0,0), new Vector2(128, 0));

jointDef.lowerAngle = -0.5f * b2_pi; // -90 degrees
jointDef.upperAngle = 0.25f * b2_pi; // 45 degrees

jointDef.enableLimit = true;
jointDef.enableMotor = true;

jointDef.maxMotorTorque = 10.0f;
jointDef.motorSpeed = 0.0f;
```

### RopeJoint

A rope joint enforces a maximum distance between two points on two bodies. It has no other effect. Warning: if you attempt to change the maximum length during the simulation you will get some non-physical behavior. A model that would allow you to dynamically modify the length would have some sponginess, so I chose not to implement it that way. See b2DistanceJoint if you want to dynamically control length.

```java
RopeJointDef jointDef = new RopeJointDef (); // has no initialize
```

### WeldJoint

A weld joint essentially glues two bodies together. A weld joint may distort somewhat because the island constraint solver is approximate.

```java
WeldJointDef jointDef = new WeldJointDef ();
jointDef.initialize(bodyA, bodyB, anchor);
```

### WheelJoint

A wheel joint. This joint provides two degrees of freedom: translation along an axis fixed in bodyA and rotation in the plane. You can use a joint limit to restrict the range of motion and a joint motor to drive the rotation or to model rotational friction. This joint is designed for vehicle suspensions.

```java
WheelJointDef jointDef = new WheelJointDef();
jointDef.maxMotorTorque = 1f;
jointDef.motorSpeed = 0f;
jointDef.dampingRatio = 1f;
jointDef.initialize(bodyA, bodyB, anchor, axis); // axis is Vector2(1,1)

WheelJoint joint = (WheelJoint) physics.createJoint(jointDef);
joint.setMotorSpeed(1f);
```

## Fixture Shapes

As mentioned previously, a fixture has a shape, density, friction and restitution attached to it.
Out of the box you can easily create boxes (as seen in the section [Static Bodies](/wiki/extensions/physics/box2d#static-bodies) section) and circle shapes (as seen in the [Dynamic Bodies](/wiki/extensions/physics/box2d#dynamic-bodies) section).

You can programatically define more complex shapes using the following classes
* ChainShape,
* EdgeShape,
* PolygonShape

However using third party tools you can simply define your shapes and import them into your game.

### Importing Complex Shapes using box2d-editor

[box2d-editor](https://github.com/julienvillegas/box2d-editor) is a free open source tool to define complex shapes and load them into your game.
An example of how to import a shape into your game using box2d-editor is available on [Libgdx.info](https://libgdxinfo.wordpress.com/box2d-importing-complex-bodies/).

Check out the [Tools section](/wiki/extensions/physics/box2d#Tools) for more tools.

In a nutshell, if you are using Box2d-editor:
* Create your shape within Box2d-editor.
* Export your scene and copy the file into your asset folder.
* Copy file BodyEditorLoader.java into your "core" module source folder.

Then in your game you can do:

```java
BodyEditorLoader loader = new BodyEditorLoader(Gdx.files.internal("box2d_scene.json"));

BodyDef bd = new BodyDef();
bd.type = BodyDef.BodyType.KinematicBody;
body = world.createBody(bd);

// 2. Create a FixtureDef, as usual.
FixtureDef fd = new FixtureDef();
fd.density = 1;
fd.friction = 0.5f;
fd.restitution = 0.3f;

// 3. Create a Body, as usual.
loader.attachFixture(body, "gear", fd, scale);
```

## Sprites and Bodies

The easiest way to manage a link between your sprites or game objects and Box2D is with Box2D’s User Data. You can set the user data to your game object and then update the object's position based on the Box2D body.

Setting a body's user data is easy

```java
body.setUserData(Object);
```

This can be set to any Java object. It is also good to create your own game actor/object class which allows you to set a reference to its physics body.

Fixtures can also have user data set to them in the same way.

```java
fixture.setUserData(Object);
```

To update all your actors/sprites you can loop through all the world's bodies easily in your game/render loop.

```java
// Create an array to be filled with the bodies
// (better don't create a new one every time though)
Array<Body> bodies = new Array<Body>();
// Now fill the array with all bodies
world.getBodies(bodies);

for (Body b : bodies) {
    // Get the body's user data - in this example, our user
    // data is an instance of the Entity class
    Entity e = (Entity) b.getUserData();

    if (e != null) {
        // Update the entities/sprites position and angle
        e.setPosition(b.getPosition().x, b.getPosition().y);
        // We need to convert our angle from radians to degrees
        e.setRotation(MathUtils.radiansToDegrees * b.getAngle());
    }
}
```

Then render your sprites using a libGDX `SpriteBatch` as usual.

## Sensors
Sensors are Bodies that do not produce automatic responses during a collision (such as applying force). This is useful when one needs to be in complete control of what happens when two shapes collide.
For example, think of a drone that has some kind of circular distance of sight. This body should follow the drone but shouldn't have a physical reaction to it, or any other bodies. It should detect when some target is inside it's shape.

To configure a body to be a sensor, set the 'isSensor' flag to true. An example would be:

```java
//At the definition of the Fixture
fixtureDef.isSensor = true;
```

In order to listen to this sensor contact, we need to implement the ContactListener interface methods.

## Contact Listeners
The Contact Listeners listen for collisions events on a specific fixture. The methods are passed a Contact object, which contain information about the two bodies involved.
The beginContact method is called when the object overlaps another. When the objects are no longer colliding, the endContact method is called.

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

This class needs to be set as the world's contact listener in the screen's show() or init() method.

```java
world.setContactListener(ListenerClass);
```

We might get information about the bodies from the contact fixtures.
Depending on the application design, the Entity class should be referenced in the Body or Fixture user data, so we can use it from the Contact and make some changes (e.g. change the player health).

## Resources

There are a lot of really good Box2D resources out there and most of the code can be easily converted to libgdx.

  * A basic implementation and code sample for Box2D with Scene2D is also available on [LibGDX.info](https://libgdxinfo.wordpress.com/box2d-basic/).
  * [Box2D documentation](https://box2d.org/documentation/) and [Discord](https://discord.com/invite/NKYgCBP) are a great place to find help.
  * A really good [tutorial series on Box2D](https://www.iforce2d.net/b2dtut/). Covers a lot of different problems which you will more than likely run across in your game development.

## Tools

The following is a list of tools for use with box2d and libgdx:

### Free Open Source

  * [Physics Body Editor](https://github.com/julienvillegas/box2d-editor)

Code sample available on [https://libgdxinfo.wordpress.com](https://libgdxinfo.wordpress.com/box2d-importing-complex-bodies//)

### Commercial

  * [RUBE](https://www.iforce2d.net/rube/) editor for creating box2d worlds. Use[RubeLoader](https://github.com/indiumindeed/RubeLoader) for loading RUBE data into libgdx.
  * [PhysicsEditor](https://www.codeandweb.com/physicseditor)
