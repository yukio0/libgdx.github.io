---
title: ファイルハンドル
---
* [はじめに](#introduction)
* [各プラットフォームのファイルシステム](#platform-filesystems)
* [ファイル（ストレージ）種別](#file-storage-types)
* [ストレージの利用可否とパスの確認](#checking-storage-availability-and-paths)
* [FileHandleの取得](#obtaining-filehandles)
* [ファイル一覧とプロパティの確認](#listing-and-checking-properties-of-files)
* [エラーハンドリング](#error-handling)
* [ファイルの読み込み](#reading-from-a-file)
* [ファイルへの書き込み](#writing-to-a-file)
* [ファイル／ディレクトリの削除・コピー・リネーム・移動](#deleting-copying-renaming-and-moving-filesdirectories)


## はじめに {#introduction}
libGDXアプリケーションは、次の4系統のプラットフォームで動作します。デスクトップ（Windows／Linux／macOS／Headless）、Android、iOS、そしてJavaScript/WebGL対応ブラウザです。これらのプラットフォームは、ファイルI/Oの扱いがそれぞれ少しずつ異なります。libGDXの[Files](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/Files.html) [(コード)](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/Files.java)モジュールは、全プラットフォーム共通のインターフェースを提供し、次のことができます。

  * ファイルを読む
  * ファイルに書く
  * ファイルをコピーする
  * ファイルを移動する
  * ファイルを削除する
  * ファイル／ディレクトリを列挙する
  * ファイル／ディレクトリの存在を確認する

libGDXのファイル処理の詳細に入る前に、対応プラットフォーム間のファイルシステムの違いを押さえておきましょう。

## 各プラットフォームのファイルシステム {#platform-filesystems}
### デスクトップ（Windows／Linux／macOS／Headless）
デスクトップOSでは、ファイルシステムはひとつの大きなメモリの塊です。ファイルは、カレントワーキングディレクトリ（アプリを実行したディレクトリ）からの相対パス、または絶対パスで参照できます。ファイル権限をいったん無視すれば、通常ファイルやディレクトリは、ほとんどのアプリケーションから読み書き可能です。

### Android
Androidはもう少し複雑です。ファイルはアプリの[APK](https://ja.wikipedia.org/wiki/APK_(%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E5%BD%A2%E5%BC%8F))内に、リソース（resource）またはアセット（asset）として格納できます。これらは読み取り専用です。libGDXは、バイトストリームへ生でアクセスでき、従来のファイルシステムに近いという理由から、[アセット機構](https://developer.android.com/reference/android/content/res/AssetManager)のみを使用します。[リソース](https://developer.android.com/guide/topics/resources/providing-resources)は一般的なAndroidアプリには向いていますが、ゲーム用途では問題を生みやすいです（例：Androidがロード時に画像を自動リサイズするなど）。

アセットはプロジェクトの`assets`ディレクトリに置かれ、アプリをデプロイするとAPKに自動でパッケージされます。これらは`Gdx.files.internal`からアクセスできます。ここは読み取り専用ディレクトリであり、Androidドキュメントで言うところの「internal（内部ストレージ）」とは別物なので混同しないでください。Android上の他のアプリから、これらのファイルへアクセスすることはできません。

ファイルは、Androidドキュメントで言うところの[内部ストレージ](https://developer.android.com/training/data-storage)（libGDXでは `Gdx.files.local`）にも保存できます。ここは読み書き可能です。インストールされた各アプリは専用の内部ストレージディレクトリを持ち、そのディレクトリはそのアプリからのみアクセスできます。これはアプリ専用の「プライベートな作業領域」と考えると分かりやすいでしょう。

さらに、外部ストレージにも保存できます。libGDXでは`Gdx.files.external`でアクセスします。外部ファイルの扱いはAndroidの歴史の中で何度も変わってきたため、libGDXでは次のようになっています。

* libGDX 1.9.11までは、Androidの外部ストレージディレクトリを使用します。Android 4.3まではSDカードディレクトリ（常に利用可能とは限らない）で、それ以降は仮想（エミュレート）SDカードディレクトリです。これらへアクセスするには、AndroidManifest.xmlに権限を追加する必要があります。詳しくは[権限](/wiki/app/starter-classes-and-configuration#permissions)を参照してください。Android 6以降は実行時権限も必要になります。さらにAndroid 11以降では（Play Storeに公開する普通のアプリの場合）このディレクトリへのアクセスは完全に禁止されています。
* libGDX 1.9.12 以降は、アプリ固有の外部ストレージディレクトリを使用します。このディレクトリ（`Android/data/data/your_package_id/` に配置されます）は、追加の権限や設定変更なしにアプリから読み書きできます。Android 10までは他のアプリ（ファイルマネージャ等）もアクセスできますが、Android 11以降はUSB経由でのみアクセス可能になります。※ユーザーがアプリをアンインストールすると、ここに保存したデータは（ユーザーが事前に別の場所へコピーしていない限り）削除されます。

アプリ固有の外部ストレージはゲーム起動時に利用できるよう初期化されるため、Androidは空のディレクトリを作成します。外部ファイルを使わず、この挙動を抑止したい場合は、`AndroidApplication#createFiles` 内で `AndroidFiles` の生成をオーバーライドして回避できます（1.9.14以降）。

```java
	protected AndroidFiles createFiles() {
		this.getFilesDir(); // Androidのバグ #10515463 の回避策
		return new DefaultAndroidFiles(this.getAssets(), this, false);
	}
```

### iOS
iOSでは、すべてのファイル種別が利用可能です。

### Javascript/WebGL
素のJavaScript/WebGLアプリには、従来型のファイルシステムという概念がありません。代わりに、画像などのアセットは、1つ以上のサーバ上のファイルを指すURLで参照します。近年のブラウザは[ローカルストレージ](https://web.archive.org/web/20240621005117/http://diveintohtml5.info/storage.html)もサポートしており、従来の読み書き型ファイルシステムに近い仕組みです。ただし、ローカルストレージはデフォルトで使える容量が比較的小さく、標準化されておらず、クォータ（上限）を正確に問い合わせる良い方法がありません。このため、JSプラットフォームでローカルデータを永続的に書き込む手段としては、現状ではpreferences APIが実質的に唯一の方法です。

libGDXは内部で工夫をして、読み取り専用のファイルシステム抽象化を提供しています。

## ファイル（ストレージ）種別 {#file-storage-types}
libGDXにおけるファイルは、[FileHandle](https://github.com/libgdx/libgdx/tree/master/gdx/src/com/badlogic/gdx/files/FileHandle.java)クラスのインスタンスとして表現されます。FileHandleにはタイプがあり、ファイルがどこに存在するかを定義します。次の表は、各プラットフォームでの利用可否と配置場所を示しています。

| *タイプ* | *説明（ファイルパスと特徴）* | *Desktop* | *Android* | *HTML5* | *iOS* |
|:------:|:--------------------------------------|:---------:|:---------:|:-------:|:-----:|
| Classpath | Classpathファイルはソースフォルダ内に直接置かれます。jarに同梱され、常に*読み取り専用*です。用途はありますが、可能なら避けるのが無難です。 | Yes | Yes | No | Yes |
| Internal | Internalファイルは、デスクトップではアプリの*ルート*または*作業（working）*ディレクトリからの相対、Androidでは*assets*ディレクトリからの相対、GWTでは`core/assets/`からの相対です。これらは*読み取り専用*です。internalに見つからない場合、Filesモジュールはクラスパスも検索します。これは、Eclipseのassetフォルダリンク機構を使う場合に必要になります（[プロジェクトの作成](/wiki/start/project-generation)を参照）。相対パス（`./` や `../`）は常にサポートされるとは限らないため使わないでください。 | Yes | Yes | Yes | Yes |
| Local | Localファイルは、デスクトップではアプリの*ルート*または*作業（working）*ディレクトリからの相対、Androidではアプリの内部（プライベート）ストレージからの相対に保存されます。なお、デスクトップではLocalとInternalはほぼ同じです。 | Yes | Yes | No | Yes |
| External| Externalファイルのパスは、デスクトップでは現在ユーザーの[ホームディレクトリ](https://www.roseindia.net/java/beginners/UserHomeExample.shtml)からの相対になります。Androidではアプリ固有の外部ストレージが使われます。 | Yes | Yes | No | Yes |
| Absolute | Absoluteファイルは完全修飾パスを指定する必要があります。<br/>*注:* 可搬性の観点から、本当に必要な場合にのみ使用してください。 | Yes | Yes | No | Yes |

AbsoluteとClasspathは、デスクトップ用エディタなど、より複雑なファイルI/O要件を持つツール用途で主に使われます。ゲーム用途では基本的に無視して構いません。通常、種別の使い分けは次の順序が目安です。

  * **Internalファイル**: アプリに同梱される全アセット（画像、音声など）はinternalファイルです。セットアップツールを使っているなら、Androidプロジェクトの`assets`フォルダに入れるだけです。
  * **Localファイル**: 小さなファイル（例：セーブデータ）を書きたい場合はlocalを使います。一般にアプリ専用の領域になります。キー／バリュー形式が欲しいなら[プリファレンス](/wiki/preferences)も検討してください。  
    ※ Androidのアプリ固有キャッシュは`../cache`でアクセスできます。ここに保存されたファイルは、アプリ設定にある「キャッシュを削除」ボタンでユーザーが消せます。
  * **Externalファイル**: 大きなファイル（例：スクリーンショット）やWebからダウンロードしたファイルを保存したい場合はexternalが候補です。外部ストレージは揮発的で、ユーザーが取り外したり、あなたが書いたファイルを削除したりできます。また自動でクリーンアップされない一方で揮発的でもあるため、通常はlocalストレージのほうが扱いやすいことが多いです。

## ストレージの利用可否とパスの確認 {#checking-storage-availability-and-paths}
ストレージ種別は、アプリが動作しているプラットフォームによって利用できない場合があります。Filesモジュールで利用可否を問い合わせられます。

```java
boolean isExtAvailable = Gdx.files.isExternalStorageAvailable();
boolean isLocAvailable = Gdx.files.isLocalStorageAvailable();
```

external/localのルートパスも取得できます。

```java
String extRoot = Gdx.files.getExternalStoragePath();
String locRoot = Gdx.files.getLocalStoragePath();
```

## FileHandleの取得 {#obtaining-filehandles}
`FileHandle`は、上で挙げた各種別を*Files*モジュールから直接呼び出して取得します。
次のコードは、internalの`myfile.txt`へのハンドルを取得します。

```java
FileHandle handle = Gdx.files.internal("data/myfile.txt");
```

[gdx-setup tool](/wiki/start/project-generation)を使っている場合、このファイルはプロジェクトの`assets`フォルダ（正確には`/assets/data`）に含まれます。デスクトップとhtmlのプロジェクトはEclipseでこのフォルダへリンクされ、Eclipseから実行すると自動的に拾われます。

```java
FileHandle handle = Gdx.files.classpath("myfile.txt");
```

`myfile.txt`は、コンパイル済みクラスがあるディレクトリ、または含まれているjarファイル内に置かれます。

```java
FileHandle handle = Gdx.files.external("myfile.txt");
```

この場合、`myfile.txt`は、デスクトップではユーザーの[ホームディレクトリ](https://ja.wikipedia.org/wiki/%E3%83%9B%E3%83%BC%E3%83%A0%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA)（Linuxなら`/home/<user>/myfile.txt`、macOSなら`/Users/<user>/myfile.txt`、Windowsなら`C:\Users\<user>\myfile.txt`）に置く必要があります。AndroidではSDカードのルートに置かれます。

```java
FileHandle handle = Gdx.files.absolute("/some_dir/subdir/myfile.txt");
```

absoluteのFileHandleでは、ファイルはフルパスが指す場所にそのまま存在していなければなりません。Windowsなら現在ドライブの`/some_dir/subdir/`、Linux／macOS／Androidなら指定した絶対パスです。

`FileHandle`インスタンスは、データの読み書きを担当するクラスのメソッドへ渡して使います。例えば`Texture`で画像を読み込む場合や、`Audio`モジュールで音声を読み込む場合などに`FileHandle`を指定します。

## ファイル一覧とプロパティの確認 {#listing-and-checking-properties-of-files}
特定のファイルが存在するか確認したり、ディレクトリ内の内容を列挙したりしたい場面があります。FileHandleには、そのためのメソッドが簡潔に用意されています。

次は、特定のファイルが存在するかどうか、またファイルが実際にはディレクトリなのかどうかを確認する例です。

```java
boolean exists = Gdx.files.external("doitexist.txt").exists();
boolean isDirectory = Gdx.files.external("test/").isDirectory();
```

ディレクトリの列挙も同じくらい簡単です。

```java
FileHandle[] files = Gdx.files.local("mylocaldir/").list();
for(FileHandle file: files) {
   // ここで必要な処理を行う
}
```

**警告**: フォルダを指定しないと、`list()`の結果は空になります。

**注**: デスクトップでは、internalディレクトリの列挙はサポートされていません。この問題を回避するには、[デプロイ前にファイル一覧を生成する](https://lyze.dev/2021/04/29/libGDX-Internal-Assets-List/)方法があります。

また、ファイルの親ディレクトリを取得したり、ディレクトリ内のファイル（いわゆる 「子」）に対するFileHandleを作ることもできます。

```java
FileHandle parent = Gdx.files.internal("data/graphics/myimage.png").parent();
FileHandle child = Gdx.files.internal("data/sounds/").child("myaudiofile.mp3");
```

`parent`は`data/graphics/`を指し、`child`は`data/sounds/myaudiofile.mp3`を指します。

FileHandleには、ファイルのさまざまな属性を確認するためのメソッドが他にも多数あります。詳しくはJavadocを参照してください。

**注**: これらの関数は、現時点ではHTML5バックエンドで未実装のものが多いです。HTML5をターゲットにする場合、これらに過度に依存しないようにしてください。

## エラーハンドリング {#error-handling}
FileHandleに対する操作は、失敗することがあります。libGDXでは、検査例外ではなく、`RuntimeException`でエラーを通知する方針を採っています。理由は、多くの場合（体感で9割くらい）、アクセスするファイルは「存在していて読めることが分かっている」ものだからです（例：アプリに同梱されたinternalファイルなど）。

## ファイルの読み込み {#reading-from-a-file}
FileHandleを取得したら、ファイルから内容を読み込めるクラス（例：画像ローダー）に渡すか、自分で読み込み処理を行うかのどちらかになります。後者はFileHandleクラスの入力メソッド群を使います。次の例は、internalファイルからテキストを読み込む例です。

```java
FileHandle file = Gdx.files.internal("myfile.txt");
String text = file.readString();
```

バイナリデータの場合も、ファイルをバイト配列に簡単に読み込めます。

```java
FileHandle file = Gdx.files.internal("myblob.bin");
byte[] bytes = file.readBytes();
```

FileHandleには他にも多数の読み込みメソッドがあります。詳しくは[Javadoc](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/files/FileHandle.html)を参照してください。

## ファイルへの書き込み {#writing-to-a-file}
読み込みと同様、FileHandleにはファイルへ書き込むためのメソッドも用意されています。ただし、ファイルへ書き込みできるのはlocal／external／absoluteのタイプだけです。文字列を書き込む例は次のとおりです。

```java
FileHandle file = Gdx.files.local("myfile.txt");
file.writeString("My god, it's full of stars", false);
```

FileHandle#writeStringの第2引数は、「追記するかどうか」を指定します。falseの場合、既存の内容は上書きされます。

もちろんバイナリデータを書き込むこともできます。

```java
FileHandle file = Gdx.files.local("myblob.bin");
file.writeBytes(new byte[] { 20, 3, -2, 10 }, false);
```

他にも、`OutputStream`を使うなど、さまざまな書き込み方法を支援するメソッドがあります。こちらも詳細はJavadocを参照してください。

## ファイル／ディレクトリの削除・コピー・リネーム・移動 {#deleting-copying-renaming-and-moving-filesdirectories}
これらの操作も、書き込み可能なファイルタイプ（local／external／absolute）でのみ実行できます。ただし、コピー元は読み取り専用のFileHandleでも構いません。例をいくつか示します。

```java
FileHandle from = Gdx.files.internal("myresource.txt");
from.copyTo(Gdx.files.external("myexternalcopy.txt"));

Gdx.files.external("myexternalcopy.txt").rename("mycopy.txt");
Gdx.files.external("mycopy.txt").moveTo(Gdx.files.local("mylocalcopy.txt"));

Gdx.files.local("mylocalcopy.txt").delete();
```

コピー元とコピー先は、ファイルでもディレクトリでも構いません。

利用できるメソッドの詳細は、[FileHandleのJavadoc](https://javadoc.io/doc/com.badlogicgames.gdx/gdx/latest/com/badlogic/gdx/files/FileHandle.html)を参照してください。
