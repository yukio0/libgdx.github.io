---
title: Bulletラッパーのセットアップ
---
Bulletラッパー（`gdx-bullet`拡張）は現在、デスクトップ/Android/iOSでサポートされています。現時点ではGWTではサポートされていません。

Bulletラッパーを使うためにプロジェクトをセットアップする最も簡単な方法は、[セットアップツール](/wiki/start/project-generation)を使うことです。ここには`gdx-bullet`拡張を含めるオプションがあります。

GradleプロジェクトにBulletラッパーを手動で追加する手順は、[こちら](/wiki/articles/dependency-management-with-gradle#bullet-gradle)にあります。

Gradleを使っていない場合は、手動で追加できます。
* プロジェクトでBullet physicsを使うには、coreプロジェクトに`gdx-bullet.jar`を追加する必要があります。もしくは、`gdx-bullet`プロジェクトを、メインプロジェクトのビルドパスに含まれるプロジェクトとして追加しても構いません。
* デスクトッププロジェクトでは、ライブラリに`gdx-bullet-natives.jar`を追加する必要があります。
* Androidプロジェクトでは、`armeabi/libgdx-bullet.so`と`armeabi-v7a/libgdx-bullet.so`をAndroidプロジェクトの`libs`フォルダへコピーする必要があります。
