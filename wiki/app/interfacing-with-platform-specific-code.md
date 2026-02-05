---
title: プラットフォーム固有コードとの連携
---
広告サービスの追加やリーダーボード機能のように、Android／iOS／デスクトップでしか利用できないプラットフォーム固有のAPIへアクセスしたくなる場面はよくあります。これは、共通のAPIインターフェースを用意し、そのインターフェースに対する「プラットフォームごとの実装」を差し替えられるようにすることで実現できます。

以下の例では、Androidでのみ利用できる、とてもシンプルなリーダーボードAPIを使うことを想定します。その他のターゲットでは、呼び出されたことをログに出すか、モックの戻り値を返すだけにします。

## 共通インターフェース

最初のステップは、APIをインターフェースとして抽象化し、**coreプロジェクト**に置くことです。

```java
public interface Leaderboard {
   public void submitScore(String user, int score);
}
```

次に、各プラットフォーム向けの実装を作り、それぞれ対応するプロジェクトに配置します。

## Android実装

Androidでは、`LeaderboardsClient#submitScore(String leaderboardId, long score);`というメソッドを提供するGoogle Play APIを呼び出したいとします。

そのため、**Androidプロジェクト**で`Leaderboard`インターフェースを実装し、次のようにプラットフォーム固有のコードを呼び出します。

```java
/** Android実装。PlayGamesに直接アクセスできる **/
public class AndroidLeaderboard implements Leaderboard {

   public void submitScore(String user, int score) {
      // ユーザー名は無視する。Google Playは現在サインインしているプレイヤーのスコアとして記録するため。
      // 詳細は https://developers.google.com/games/services/android/signin を参照。
      PlayGames.getLeaderboardsClient(activity).submitScore(getString(R.string.leaderboard_id), score);
   }
}
```

## デスクトップ実装

次のコードは**デスクトップLwjgl3プロジェクト**に置きます。

```java
/** デスクトップ実装：呼び出し内容をログに出すだけ **/
public class Lwjgl3Leaderboard implements Leaderboard {
   public void submitScore(String user, int score) {
      Gdx.app.log("Lwjgl3Leaderboard", "would have submitted score for user " + user + ": " + score);
   }
}
```

## HTML5(GWT)実装
次のコードは**HTML5プロジェクト**に置きます。

```java
/** HTML5実装：Lwjgl3Leaderboardと同じ（ログ出力のみ） **/
public class Html5Leaderboard implements Leaderboard {
   public void submitScore(String user, int score) {
      Gdx.app.log("Html5Leaderboard", "would have submitted score for user " + user + ": " + score);
   }
}
```

## coreでプラットフォーム固有実装を利用する
次に、具体的な`Leaderboard`実装を渡せるように、`ApplicationListener`にコンストラクタを用意します。

```java
public class MyGame implements ApplicationListener {
   private final Leaderboard leaderboard;

   public MyGame(Leaderboard leaderboardImpl) {
      this.leaderboard = leaderboardImpl;
   }

   // 分かりやすさのため、残りは省略
}
```

そして各[スタータークラス](/wiki/app/starter-classes-and-configuration)で、対応する`Leaderboard`実装を引数に渡して`MyGame`を生成します。たとえばデスクトップなら次のようになります。

```java
public static void main(String[] argv) {
   Lwjgl3ApplicationConfiguration config = new Lwjgl3ApplicationConfiguration();
   new Lwjgl3Application(new MyGame(new Lwjgl3Leaderboard()), config);
}
```

別の方法として、リフレクションを使ってプラットフォーム固有の実装を取得することもできます。

```java
if (Gdx.app.getType() == ApplicationType.Desktop || Gdx.app.getType() == ApplicationType.HeadlessDesktop) {
    try {
		    this.leaderboard = (Leaderboard) ClassReflection.newInstance(ClassReflection.forName("com.mygame.lwjgl3.Lwjgl3Leaderboard"));
		} catch (ReflectionException e) {
		    e.printStackTrace();
		}
}
```
