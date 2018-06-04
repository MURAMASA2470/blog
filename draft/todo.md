## 記事整理
- カテゴリーDB -> カテゴリーetc、フォルダetcへ移動？
- /env/* -> /etc/* （カテゴリーも etc に変更？）

## リリース
- lsFeed 1.0？

## リポジトリ整理
- kaze-samples を kaze 配下に移す。

## レビュー
- elecom gamepad
- hori gamepad
- dell monitor
- dell soundbar
- sony tv

## Java（Jetty -> Jettyを大がかりに整理？）
- とりあえず目次は更新した。
- 後、/entry/jetty/* を /entry/java/jetty に移すかなどを検討したい。

## Websocket の記事作成
- 構想
  - JSR-356 を使う。
  - 組込Jetty と Tomcat の両方で使えるサンプルアプリにしたい。
  - まずは Tomcat 8.5 で Websocket できるように、サンプルを用意したい。
- サンプルコード
  - 組込Jetty -> ejws（参考文献→ https://github.com/jetty-project/embedded-jetty-websocket-examples）
  - Tomcat 8.5 -> http://tomcat.apache.org/tomcat-8.5-doc/web-socket-howto.html
- chat を開始して、タブを閉じると closeになる
- chat を開始して、ずっとつないでおくと timeout がおきる。
- firefox と chrome だとタイムラグがある。
```
java.net.SocketTimeoutException: Timeout on Read
  at org.eclipse.jetty.websocket.common.io.AbstractWebSocketConnection.onReadTimeout(AbstractWebSocketConnection.java:592)
  at org.eclipse.jetty.io.AbstractConnection.onFillInterestedFailed(AbstractConnection.java:170)
  at org.eclipse.jetty.websocket.common.io.AbstractWebSocketConnection.onFillInterestedFailed(AbstractWebSocketConnection.java:538)
  at org.eclipse.jetty.io.AbstractConnection$ReadCallback.failed(AbstractConnection.java:285)
  at org.eclipse.jetty.io.FillInterest.onFail(FillInterest.java:140)
  at org.eclipse.jetty.io.AbstractEndPoint.onIdleExpired(AbstractEndPoint.java:398)
  at org.eclipse.jetty.io.IdleTimeout.checkIdleTimeout(IdleTimeout.java:166)
  at org.eclipse.jetty.io.IdleTimeout$1.run(IdleTimeout.java:50)
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
  at java.util.concurrent.FutureTask.run(FutureTask.java:266)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
  at java.lang.Thread.run(Thread.java:748)
  ```