---
title: ロギング
---
`Application`インターフェースには、どのメッセージをログに出すかを細かく制御できる、シンプルなロギング機能が用意されています。

メッセージは、通常の**情報（info）メッセージ**、例外を任意で付けられる**エラーメッセージ**、そして**デバッグメッセージ**を出力できます。

```java
Gdx.app.log("MyTag", "my informative message");
Gdx.app.error("MyTag", "my error message", exception);
Gdx.app.debug("MyTag", "my debug message");
```

デスクトップではメッセージはコンソールに出力され、Androidではlogcatに出力されます。GWTでは、ブラウザのコンソール、または`GwtApplicationConfiguration`で指定できる`TextArea`のいずれかに出力されます。

ロギングは、特定のログレベルに制限できます。

```java
Gdx.app.setLogLevel(logLevel);
```

`logLevel`は次のいずれかです。

  * `Application.LOG_DEBUG`: すべてのメッセージをログに出力します。
  * `Application.LOG_INFO`: エラーメッセージと通常の情報メッセージをログに出力します。
  * `Application.LOG_ERROR`: エラーメッセージのみをログに出力します。
  * `Application.LOG_NONE`: ログ出力を行いません。
