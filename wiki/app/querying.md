---
title: クエリ
---
`Application`インターフェースには、実行時環境の各種プロパティをクエリ（問い合わせ）するためのメソッドが用意されています。

### 実行プラットフォームの取得
実行しているプラットフォームによって、処理を分けて実装したい場合があります。`Application.getType()`メソッドは、アプリケーションが現在動作しているプラットフォームを返します。

```java
switch (Gdx.app.getType()) {
    case Android:
        // Android固有のコード
        break;
    case Desktop:
        // デスクトップ固有のコード
        break;
    case WebGl:
        // HTML5固有のコード
        break;
    default:
        // その他のプラットフォーム固有のコード
}
```

AndroidとiOSでは、現在動作しているOSのバージョンも取得できます。

```java
int androidVersion = Gdx.app.getVersion();
```

Androidでは、現在の端末のSDKレベル（APIレベル）を返します（例：Android 1.5なら3）。iOSでは、現在のOSのメジャーバージョンを返します。

### メモリ使用量
デバッグやプロファイリングの目的で、Javaヒープとネイティブヒープの両方のメモリ使用量を把握したいことがよくあります。

```java
long javaHeap = Gdx.app.getJavaHeap();
long nativeHeap = Gdx.app.getNativeHeap();
```

どちらのメソッドも、それぞれのヒープで現在使用中のバイト数を返します。
