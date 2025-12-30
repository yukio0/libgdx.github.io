---
title: "開発環境の構築"
description: "最初のlibGDXプロジェクトを立ち上げる前に、開発環境を構築する必要があります。最初のステップはIDEを選ぶことです。多くの人が選ぶIDEとしては、Android Studio、IntelliJ IDEA、Eclipseが挙げられます。"
redirect_from:
  - /dev/setup/
---

libGDXを初めて使う人は、まさにこのページが最適です。以下の手順では、最初のlibGDXプロジェクトを立ち上げる方法を説明します。

{% include setup_flowchart.html current='0' %}

libGDXを始める前に、まずIDE（統合開発環境）を準備する必要があります。IDEとは、Javaファイルを編集するためのエディタで、Javaアプリケーション開発をさまざまな面で非常に便利にしてくれます。**すでにIDEをインストールしている場合は、次の[ステップ](/wiki/start/project-generation)に進んでください。**

Javaの世界には多くのIDEがあります。それぞれに小さな長所・短所はありますが、最終的にはどれも開発の役割を果たすので、好きなものを選んでください。

## (1.) Android Studio
デスクトップだけでなくモバイルもターゲットにしたい初心者には、**Android Studioがおすすめです**。
{: .notice--info}

- JDK: Android Studioに同梱
- IDE: [Android Studio](https://developer.android.com/studio)
- Android対応: 標準でサポート
- iOS対応: [RoboVM OSS IntelliJプラグイン](https://mobivm.github.io)

## (2.) IDEA
- JDK 17または21: いくつかディストリビューションがありますが、[Adoptium](https://adoptium.net/)をおすすめします。
- IDE: [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) （Community版で十分です）
- Android対応: [Android SDK](https://developer.android.com/tools/releases/platform-tools)
- iOS対応: [RoboVM OSS IntelliJプラグイン](https://mobivm.github.io)

## (3.) Eclipse
- JDK 17または21: いくつかディストリビューションがありますが、[Adoptium](https://adoptium.net/)をおすすめします。
- IDE: [Eclipse](https://www.eclipse.org/downloads/)
- Android対応: 公式にはサポートされていませんが、[Andmore](https://projects.eclipse.org/projects/tools.andmore)を使うか、古いバージョンの[ADT](https://marketplace.eclipse.org/content/android-development-tools-eclipse)を試してみるとうまくいくかもしれません。
- iOS対応: [RoboVM OSS Eclipseプラグイン](https://mobivm.github.io)

## (4.) Other IDEs
もちろん、NetBeansなどのJava用IDEやVisual Studio Codeなどを使うこともできます。ただし、libGDXコミュニティではあまり一般的でないため、IDE固有の問題が発生した場合はサポートを受けにくいかもしれません！
{: .notice--info}
- [NetBeans](https://netbeans.apache.org/download/index.html)はNetBeans Gradleプラグインが必要。AndroidおよびiOSは公式サポートされていません。
- Visual Studio CodeはJava対応の拡張が必要です。[Coding Pack for Java](https://code.visualstudio.com/docs/java/java-tutorial#_coding-pack-for-java)を参照してください。AndroidおよびiOSは公式サポートされていません。
- AIDEはAndroid 10以前の端末でのAndroid開発のみ対応しています。libGDXのJARファイルは[こちら](https://repo1.maven.org/maven2/com/badlogicgames/gdx/)から入手可能です。

## (5.) IDEを使わない場合
IDEをまったく使わず、Notepadや[Vim](https://www.vim.org)のようなシンプルなエディタだけでlibGDXアプリを開発することも可能です。しかし、IDEはコード補完やエラーチェックなど非常に便利な機能を提供するため、IDEを使わないことは推奨**しません**。どうしてもIDEを使わずに開発したい場合には、libGDXアプリはGradleプロジェクトなので、コマンドラインでビルドや実行が可能です。
{: .notice--info}

- JDK 17または21: いくつかディストリビューションがありますが、[Adoptium](https://adoptium.net/)をおすすめします。
- Android対応: [Android SDK](https://developer.android.com/tools/releases/platform-tools)
- ANDROID_HOME環境変数を設定するか、gradle.propertiesを利用してください。

<br/>

**これで開発環境が準備できたので、最初のlibGDXプロジェクトを作成できます。libGDXにはプロジェクト生成ツールが用意されており、必要なファイルを自動で作成してくれます。使い方は[こちら](/wiki/start/project-generation)をご覧ください。**
