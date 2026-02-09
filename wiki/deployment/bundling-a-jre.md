---
title: JREのバンドル
---
Javaアプリを実行するには、Java Runtime Environment（JRE）が必要です。通常はユーザーが自分でインストールしており、アプリを起動するときにはすでに利用できる状態でしょう。しかし、残念ながら、Javaが入っていないユーザーもいますし、JREの違いによってアプリで問題が起きることもあります。こうした問題は、ユーザーが状況を説明するのが難しいうえに、さらに厄介なことに自力で修正するのも難しかったりします。また、アプリ側が「最低でもこのバージョンのJREが必要」といった要件を持つ場合もあります。

解決策は、アプリにJREをバンドル（同梱）することです。こうすれば、ユーザーが実際に動かす実行環境をこちらで正確に把握でき、ユーザー側のトラブルも減ります。さらに、ユーザーはJVMをインストールしなくても済みます。

## パッケージング
JREを同梱するためのツールはいくつかあります。

### [Construo](https://github.com/fourlastor-alexandria/construo?tab=readme-ov-file#construo)
Windows／Linux／Mac向けに、libGDXアプリを「最小化してパッケージング」し、配布できる形にするモダンな方法です。対象プラットフォームに応じたGradleコマンドを呼び出すだけでOKです。これらのコマンドは、どのOSからでも実行できます。

**Mac M1** lwjgl3:packageMacM1<br>
**Mac OSX** lwjgl3:packageMacX64<br>
**Linux** lwjgl3:packageLinuxX64<br>
**Windows** lwjgl3:packageWinX64<br>

これにより、ターゲットプラットフォーム向けのゲーム本体と、最小化されたJREを含むzipファイルが`lwjgl3/build/construo/dist`に作成されます。詳しくは、GDX-Liftoffの動画の[このセクション](https://www.youtube.com/watch?v=VF6N_X_oWr0&t=1088s)を参照してください。

### [Graal Native Image](https://www.graalvm.org/latest/reference-manual/native-image/)
Graal Native Imageは、Javaコードを事前（AOT: ahead-of-time）にコンパイルして、ネイティブ実行ファイルを作る方法です。ゲームにJREを同梱する必要がある他の手法とは異なります。ネイティブ実行ファイルはサイズがずっと小さく、実行時に必要なリソースも少なく、起動もほぼ一瞬になります。有効化するには、`gradle.properties`に次を設定します。

```
enableGraalNative=true
```

また、Graal JDKをインストールしておく必要があります。なお、リフレクションやリソース利用に関して追加の対応が必要になりやすく、「何もしなくてもそのまま動く」ことは期待しないほうがよいです。詳しくは[公式ドキュメント](https://graalvm.github.io/native-build-tools/latest/gradle-plugin.html)を参照してください。

### [Packr](https://github.com/libgdx/packr)
libGDXチームが作成・メンテナンスしているパッケージングツールです。使ってみたい場合は、[リポジトリ](https://github.com/libgdx/packr#usage)を参照してください。

### [Parcl](https://github.com/mini2Dx/parcl)
launch4jと似た処理を行うGradleプラグインです。使い方は[README](https://github.com/mini2Dx/parcl#how-to-use)を参照してください。

### [jpackage](https://docs.oracle.com/en/java/javase/14/jpackage/packaging-overview.html#GUID-C1027043-587D-418D-8188-EF8F44A4C06A)

jpackageは、[JEP-343](https://openjdk.java.net/jeps/343)で導入された、Windows／MacOS／Linux向けにネイティブパッケージングを行うツールです。同梱したJREを使ってアプリを起動するEXEを作成できます。大きな欠点として、ターゲットごとの実行ファイル（Windows用、Linux用、Mac用）を作るためには、それぞれのターゲットOS上で直接実行する必要があります。

使い方の詳細は[このガイド](https://github.com/raeleus/skin-composer/wiki/libGDX-and-JPackage)を参照してください。動画版は[こちら](https://www.youtube.com/watch?v=R7CMXeQ11GM)にあります。なお、これらのガイドは古く、現在は推奨されません。

### [Jpackage Gradle Plugin](https://github.com/petr-panteleyev/jpackage-gradle-plugin)
最新のGradle標準に沿う形で、jpackageを便利にラップしたGradleプラグインです。使い方は[README](https://github.com/petr-panteleyev/jpackage-gradle-plugin/blob/master/README.md)を参照してください。

### [launch4j](http://launch4j.sourceforge.net/)
_-- どうやらメンテナンスされていないようです --_

## MacOS固有の注意点

MacOS向けにも配布する予定なら、公証（[notarization](https://developer.apple.com/documentation/xcode/notarizing_macos_software_before_distribution)）（MacOS 10.15+）が問題になる場合があります。libGDXアプリを公証する方法については、[こちら](https://www.joelotter.com/2020/08/14/macos-java-notarization.html)を参照してください。

## サイズ削減

JREから不要なファイルやクラスを削除することで、サイズを小さくできます。以下はWindowsのJREから削除するファイルの一覧例です。他のプラットフォームも概ね似ていますが、プラットフォームによっては必要なクラスが異なることがあります（例：Linuxで`java.util.preferences`を使うには、xml関連クラスが必要です）。このリストではSwingは残しています。Swingが不要なら、さらにサイズを削れます。

```
**.diz
**.exe except javaw.exe
bin\client\
lib\applet\
lib\charsets.jar
lib\ext\localedata.jar
lib\management\
lib\management-agent.jar
lib\zi\
lib\rt.jar\com\sun\org\
lib\rt.jar\com\sun\xml\
lib\rt.jar\com\sun\corba\
lib\rt.jar\com\sun\media\
lib\rt.jar\com\sun\jndi\
lib\rt.jar\com\sun\imageio\
lib\rt.jar\com\sun\jmx\
lib\rt.jar\com\sun\rowset\
lib\rt.jar\com\sun\java\util\
lib\rt.jar\javax\imageio\
lib\rt.jar\javax\management\
lib\rt.jar\javax\print\
lib\rt.jar\javax\naming\
lib\rt.jar\javax\sound\
lib\rt.jar\javax\sql\
lib\rt.jar\javax\xml\
lib\rt.jar\javax\swing\plaf\nimbus\
lib\rt.jar\javax\swing\text\html\
lib\rt.jar\org\
lib\rt.jar\sun\applet\
lib\rt.jar\sun\management\
lib\rt.jar\sun\rmi\
lib\rt.jar\sun\security\jgss\
lib\rt.jar\sun\security\krb5\
lib\rt.jar\sun\security\tools\
lib\resources.jar\com\sun\corba\
lib\resources.jar\com\sun\imageio\
lib\resources.jar\com\sun\jndi\
lib\resources.jar\com\sun\org\
lib\resources.jar\com\sun\rowset\
lib\resources.jar\com\sun\servicetag\
lib\resources.jar\com\sun\xml\
lib\jsse.jar\sun\security\ssl\
```

このリストを作るために、ファイルやJARをサイズの大きい順に見ていきました。そして「不要そうに見える」大きなファイルから削除し、アプリを実行して問題なく動くことを確認しました。

このリストを適用すると、JREサイズは約36MBになります。なお、起動を速くするためにJREのJARは圧縮されていません。JRE全体をzipにすると、サイズは約13.5MBまで減ります。さらにrt.jarからSwing関連パッケージも削除すると、zip後のサイズは約9.8MBまで下がります。
