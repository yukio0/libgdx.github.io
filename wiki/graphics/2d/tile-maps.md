---
title: タイルマップ
---
# マップ

libGDXには汎用的なマップAPIが用意されています。マップ関連のクラスはすべて、[com.badlogic.gdx.maps](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/package-use.html) [(コード)](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/maps)パッケージにあります。ルートのパッケージには基本クラスが置かれており、サブパッケージにはタイルマップやその他の形式のマップ向けに特化した実装が含まれます。

## 基本クラス
基本クラス群は汎用的に作られており、タイルマップだけでなく、あらゆる2Dマップ形式をサポートできるようになっています。

マップは複数のレイヤから構成されます。レイヤは複数のオブジェクトを含みます。マップ、レイヤ、オブジェクトはそれぞれプロパティを持ち、その内容は読み込んだマップ形式に依存します。形式によっては、専用のマップ、レイヤ、オブジェクト実装が用意されていることもあります。これについては後ほど説明します。基本クラスのクラス階層は次のとおりです。

![images/maps-api.png](/assets/wiki/images/maps-api.png)

### プロパティ

マップ、レイヤ、オブジェクトのプロパティは[MapProperties](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/MapProperties.html)[(ソース)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/MapProperties.java)で表されます。
このクラスは本質的にはハッシュマップで、キーは文字列、値は任意の型を取れます。

マップ、レイヤ、オブジェクトで利用できるキー／値の組は読み込んだフォーマットによって異なります。プロパティにアクセスするには、単に次のように書けます。

```java
map.getProperties().get("custom-property", String.class);
layer.getProperties().get("another-property", Float.class);
object.getProperties().get("foo", Boolean.class);
```

対応しているエディタの多くでは、マップ、レイヤ、オブジェクトにこうしたプロパティを設定できます。これらのプロパティが具体的にどの型になるかはフォーマット依存です。迷った場合は、いずれかのマップローダーでマップをlibGDXアプリケーションに読み込み、目的のオブジェクトのプロパティを確認してみてください。

### マップレイヤ

マップ内のレイヤは順序付けられており、インデックス0から番号が振られます。マップのレイヤには次のようにアクセスできます。

```java
MapLayer layer = map.getLayers().get(0);
```

名前でレイヤを検索することもできます。

```java
MapLayer layer = map.getLayers().get("my-layer");
```

これらのgetterメソッドは常にMapLayerを返します。一部のレイヤは特殊化されていて、より多くの機能を提供している場合があります。その場合は、単にキャストすれば扱えます。

```java
TiledMapTileLayer tiledLayer = (TiledMapTileLayer)map.getLayers().get(0);
```

レイヤには対応しているすべてのマップ形式で共通化するようにしている属性がいくつかあります。

```java
String name = layer.getName();
float opacity = layer.getOpacity();
boolean isVisible = layer.isVisible();
```

これらは変更することもでき、レイヤがどのように描画されるかに影響する場合があります。

この共通化された属性に加えて、前述のとおり、より汎用的なプロパティにもアクセスできます。

レイヤ内のオブジェクトを取得するには、次のように呼び出します。

```java
MapObjects objects = layer.getObjects();
```

[MapObjects](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/MapObjects.html) [(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/MapObjects.java)のインスタンスを使うと、名前、インデックス、型でオブジェクトを取得できます。また、実行中にオブジェクトを追加したり削除したりすることも可能です。

### マップオブジェクト

APIには、すでにいくつかの特殊なマップオブジェクトが用意されています。例えば、
[CircleMapObject](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/objects/CircleMapObject.html) [(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/objects/CircleMapObject.java)、
[PolygonMapObject](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/objects/PolygonMapObject.html) [(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/objects/PolygonMapObject.java) 
などがあります。

マップフォーマットに対応したローダーは、これらのオブジェクトを解析し、それぞれ適切な
[マップレイヤ](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/MapLayer.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/MapLayer.java)
に格納します。

対応しているすべてのフォーマットに対して、各オブジェクトから共通化された属性を抽出するようにしています。

```java
String name = object.getName();
float opacity = object.getOpacity();
boolean isVisible = object.isVisible();
Color color = object.getColor();
```

`PolygonMapObject`のような特殊なマップオブジェクトは、追加の属性を持つ場合もあります。例えば次のようなものです。

```java
Polygon poly = polyObject.getPolygon();
```

これらの属性のいずれかを変更すると、オブジェクトがどのように描画されるかに影響することがあります。

マップやレイヤの場合と同様に、前述のとおり、より汎用的なプロパティにもアクセスできます。

*注意：* タイルマップのタイルは、マップオブジェクトとして保存されません。こうした要素をより効率的に保持するための、専用のレイヤ実装が用意されています（後述）。ここで説明したオブジェクトは一般に、トリガー領域、スポーン地点、当たり判定形状などを定義するために使われます。

### マップレンダラー

[マップレンダラー](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/MapRenderer.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/MapRenderer.java) インターフェースはマップのレイヤやオブジェクトを描画するためのメソッドを定義しています。

描画を始める前に、マップに対してビューを設定する必要があります。ビューは覗き込む窓のようなものだと考えてください。これを行う最も簡単な方法は使用する正射影カメラ（OrthographicCamera）をマップレンダラーに渡すことです。

```java
mapRenderer.setView(camera);
```

あるいは、投影行列と表示範囲（ビューの境界）を手動で指定することもできます。

```java
mapRenderer.setView(projectionMatrix, startX, startY, endx, endY);
```

表示範囲はx/y平面上で指定し、y軸は上向きです。使用する単位は、読み込んだマップおよびそのフォーマットに依存します。

マップの全レイヤを描画するには、次のように呼び出すだけです。

```java
mapRenderer.render();
```

どのレイヤを描画するかをより細かく制御したい場合は、描画したいレイヤのインデックスを指定できます。たとえば3つのレイヤ（背景レイヤが2つ、前景レイヤが1つ）を持っていて、背景と前景の間に自前のスプライトを描画したいなら、次のようにできます。

```java
int[] backgroundLayers = { 0, 1 }; // 毎フレームアロケートしないこと！
int[] foregroundLayers = { 2 };    // 毎フレームアロケートしないこと！
mapRenderer.render(backgroundLayers);
renderMyCustomSprites();
mapRenderer.render(foregroundLayers);
```

レイヤを個別に描画し、さらにレイヤごとにビューを変更することで、パララックス（視差）効果を実現することもできます。

## タイルマップ
タイルを含むレイヤを持つマップは[com.badlogic.gdx.maps.tiled](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/maps/tiled)パッケージ内のクラスで扱います。このパッケージには、複数のフォーマットに対応したローダーが含まれています。

タイルマップは
[TiledMap](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMap.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMap.java)クラスのインスタンスとして読み込まれます。`TiledMap`は汎用の`Map`クラスのサブクラスで、追加のメソッドや属性を備えています。

### タイルマップのレイヤ
タイルを含むレイヤは
[TiledMapTileLayer](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMapTileLayer.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMapTileLayer.java)のインスタンスとして保持されます。タイルにアクセスするにはキャストが必要です。

```java
TiledMap tiledMap = loadMap(); // これについては下で説明
TiledMapTileLayer layer = (TiledMapTileLayer)tiledMap.getLayers().get(0); // インデックありのレイヤがタイルを含む前提
```

`TiledMapTileLayer`は汎用の`MapLayer`と同じ属性（例：プロパティ、オブジェクトなど）をすべて持っています。

それに加えて、`TiledMapTileLayer`には、2次元配列として[TiledMapTileLayer.Cell](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMapTileLayer.Cell.html) [(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMapTileLayer.java#L89)のインスタンスが格納されています。

セルにアクセスするには、タイルレイヤから次のように取得します。

```java
Cell cell = tileLayer.getCell(column, row);
```

ここでcolumnとrowはセルの位置を表し、いずれも整数のインデックスです。タイルはy軸が上向きの座標系にある想定です。マップの左下のタイルは(0,0)、右上のタイルは(tileLayer.getWidth()-1, tileLayer.getHeight()-1)の位置になります。

その位置にタイルが存在しない場合、またはcolumn、rowが範囲外の場合はnullが返されます。

レイヤー内の横方向、縦方向のタイル数は、次のように取得できます。

```java
int columns = tileLayer.getWidth();
int rows = tileLayer.getHeight();
```

### セル
セルは
[TiledMapTile](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMapTile.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMapTile.java)のコンテナ（入れ物）です。セル自体はタイルへの参照を保持しており、さらに描画時にそのタイルを回転させるか、反転させるかを指定する属性も持っています。

タイルは通常、複数のセルから共有されます。

### タイルセットとタイル
タイルマップは1つ以上の
[TiledMapTileSet](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMapTileSet.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMapTileSet.java)インスタンスを含みます。タイルセットは複数の[TiledMapTile](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/TiledMapTile.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/TiledMapTile.java)インスタンスを含みます。タイルには[複数の実装](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/maps/tiled/tiles)（例：静的タイル、アニメーションタイルなど）があり、特別な目的のために独自の実装を作ることもできます。

タイルレイヤー内のセルはこれらのタイルを参照します。1つのレイヤー内のセルは複数のタイルセットに含まれるタイルを参照することもできます。ただし、テクスチャの切り替え回数を減らすため、1レイヤーにつき1タイルセットに統一することが推奨されます。

### タイルマップの描画
タイルマップとそのレイヤを描画するには、[専用のマップレンダラー実装](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/maps/tiled/renderers)が必要です。直交や見下ろしのマップには
[OrthogonalTiledMapRenderer](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/renderers/OrthogonalTiledMapRenderer.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/renderers/OrthogonalTiledMapRenderer.java)を使い、アイソメトリック（isometric）マップには、
[IsometricTiledMapRenderer](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/maps/tiled/renderers/IsometricTiledMapRenderer.html)
[(コード)](https://github.com/libgdx/libgdx/blob/master/gdx/src/com/badlogic/gdx/maps/tiled/renderers/IsometricTiledMapRenderer.java)を使います。このパッケージ内のその他のレンダラーは実験的なものなので、現時点では使用を推奨しません。

このようなレンダラーの作成は次のように行います。

```java
float unitScale = 1 / 16f;
OrthogonalTiledMapRenderer renderer = new OrthogonalTiledMapRenderer(map, unitScale);
```

レンダラーはコンストラクタで渡したマップだけを描画できます。この結びつきにより、レンダラーはその特定のマップに対して最適化を行い、その結果をキャッシュできます。

`unitScale`は「何ピクセルがワールドの1単位に相当するか」をレンダラーに伝えます。上の例では16ピクセルが1単位です。もし1ピクセルを1単位に対応させたいなら、`unitScale`は1にする必要があり、後は同様です。

unitScaleは描画時の座標系とゲーム世界の座標系を結びつけるための方法です。

小さな例として、タイルが32x32ピクセルのタイルマップを考えます。ゲーム世界の表現では、これを1x1単位の正方形に対応させたいとします。この場合、`unitScale`に1/32fを指定します。さらにカメラも同じ単位スケールで動作するように設定できます。たとえば画面上にマップの30x20タイルを表示したいなら、カメラは次のように作れます。

```java
OrthographicCamera camera = new OrthographicCamera();
camera.setToOrtho(false, 30, 20);
```

アイソメトリックマップでも考え方は同じで、`IsometricTiledMapRenderer`を作るだけです。

```
renderer = new IsometricTiledMapRenderer(isoMap, 1 / 32f);
```

ここでも、（アイソメトリックのタイルマップである）マップとunitScaleを指定する必要があります。

*注意：* アイソメトリック用レンダラーは実験的なものです。自己責任で使用し、問題を見つけた場合は報告してください。性能面では、モバイル端末でアイソメトリックマップを描画するのは非常にコストが高く、すべてのタイルでブレンド処理が必要になります。

### Loading TMX/Tiled maps
![images/tile-maps2.png](/assets/wiki/images/tile-maps2.png)

[Tiled](https://www.mapeditor.org/) is a generic tile map editor for Windows/Linux/Mac OS X that allows you to create tile layers as well as object layers, containing arbitrary shapes for trigger areas and other purposes. libGDX provides a loader to read files generated by Tiled.

To load a Tiled map you have two options: either load it directly or via the AssetManager. The first option works like this:

```java
TiledMap map = new TmxMapLoader().load("level1.tmx");
```

This will load the file called `level1.tmx` from the internal file storage (the assets directory). If you want to load a file using a different file type, you have to supply a FileHandleResolver in the constructor of the TmxMapLoader.

```java
TiledMap map = new TmxMapLoader(new ExternalFileHandleResolver()).load("level1.tmx");
```

We chose this mechanism as the TmxMapLoader can also be used with the AssetManager class, where FileHandleResolvers rule the earth. To load a TMX map via the AssetManager, you can do the following:

```java
// only needed once
assetManager.setLoader(TiledMap.class, new TmxMapLoader(new InternalFileHandleResolver()));
assetManager.load("level1.tmx", TiledMap.class);

// once the asset manager is done loading
TiledMap map = assetManager.get("level1.tmx");
```

Once loaded you can treat the map just like an other TiledMap.

*Note* if you load your TMX map directly, you are responsible for calling `TiledMap#dispose()` once you no longer need it. This call will dispose of any textures loaded for the map.  
*Note* if you want to use TMX maps with the GWT backend, you need to make sure the map is saved with pure base64 encoding. The compressed TMX formats will not work due to limitations in GWT.  
*Note* libGDX does not support infinite size for TMX maps (see [#5764](https://github.com/libgdx/libgdx/issues/5764)). Use only size-limited TMX maps.

### Loading TiledMapPacker Atlas TMX/TMJ Tiled maps
The libGDX TiledMapPacker and AtlasTmxMapLoader have been around for years, but with recent updates it's been expanded to include
more feature's and well as better documentation on how to use it.

Maps processed by TiledMapPacker must be loaded using the AtlasTmxMapLoader or AtlasTmjMapLoader classes instead of the standard map loaders.
These specialized loaders recognize the extra atlas property embedded in the map file, ensuring tilesets, image layers, and collection-of-images tilesets are properly loaded from the generated TextureAtlas.

Using these loaders reduces draw calls, minimizes texture binds, and improves rendering efficiency. Especially for maps which heavily rely on image layers and multiple tilesets.

See the full [TiledMapPacker documentation](/wiki/tools/tiled-map-packer) for usage instructions.

### Loading Tide maps
![images/tile-maps3.png](/assets/wiki/images/tile-maps3.png)

[Tide](https://colinvella.github.io/tIDE/) is another general purpose tile map editor, available for Windows only. libGDX provides a loader for the format output by Tide.

As with TMX files, you can either load a Tide map directly or through the asset manager:

```java
// direct loading
map = new TideMapLoader().load("level1.tide");

// asset manager loading
assetManager.setLoader(TiledMap.class, new TideMapLoader(new InternalFileHandleResolver()));
assetManager.load("level1.tide", TiledMap.class);
```

*Note* if you load your Tide map directly, you are responsible for calling TiledMap#dispose() once you no longer need it. This call will dispose of any textures loaded for the map.

## Performance considerations
While we try to make the renderers as fast as possible, there are a few things you can consider to boost rendering performance.

  * Only use tiles from a single tile set in a layer. This will reduce texture binding.
  * Mark tiles that do not need blending as opaque. At the moment you can only do this programmatically, we will provide ways to do it in the editor or automatically.
  * Do not go overboard with the number of layers.

## Examples
  * [Simple platformer using a TMX map](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/superkoalio/SuperKoalio.java)
  * [Programmatic creation of a TiledMap](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/bench/TiledMapBench.java)
  * [Tile map asset manager loading/rendering](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/TiledMapAssetManagerTest.java)
  * [Tile map direct loading/rendering](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/TiledMapDirectLoaderTest.java)
  * [Tide map asset manager loading/rendering](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/TideMapAssetManagerTest.java)
  * [Tide map direct loading/rendering](https://github.com/libgdx/libgdx/blob/master/tests/gdx-tests/src/com/badlogic/gdx/tests/TideMapDirectLoaderTest.java)
