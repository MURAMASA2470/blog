---
Title: WebSocket：エコーアプリのJava開発
Category:
- Java
Date: 2018-02-06T19:01:40+09:00
URL: https://web-dev.hatenablog.com/entry/java/websocket/echo/dev-java
EditURL: https://blog.hatena.ne.jp/mamorums/web-dev.hatenablog.com/atom/entry/8599973812344377668
---

Java の WebSocket を使って、エコーアプリを作ってみることにしました。

今回は、

1. アプリの概要
2. プロジェクトの作成
3. サーバサイドの Java開発

について書いていきます。


## 1. アプリの概要
ブラウザからメッセージを送信して、サーバーがメッセージをそのままクライアントに返す感じのアプリです。

画面のイメージは、[動作確認の記事](/entry/java/websocket/echo/check) に掲載しています。


## 2. プロジェクトの作成
Maven プロジェクトとして、以下のディレクトリ階層を作成します。

```
ws-echo
  - src
    - main
      - java（Javaコードを保存）
      - webapp（静的コンテンツを保存）
```

ルートディレクトリは、`ws-echo` という名前にしてみました。あと、ルートディレクトリ直下に `pom.xml` を作成しました。

`ws-echo/pom.xml`

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.domain</groupId>
  <artifactId>ws-echo</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <dependencies>
    <dependency>
      <groupId>org.eclipse.jetty.websocket</groupId>
      <artifactId>javax-websocket-server-impl</artifactId>
      <version>9.4.6.v20170531</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>9</maven.compiler.source>
    <maven.compiler.target>9</maven.compiler.target>
  </properties>

  <build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
      <plugin>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.1.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

組込みJetty でも動かせるように、依存性に `javax-websocket-server-impl` を追加しています。Tomcat にも配備したいので、パッケージは `war` にしています。依存性のスコープは `provided` で、`war` に含めないようにしています。


## 3. Java資源の作成
`javax.websocket.*` を使って、サーバサイドのプログラムを作成します。

`ws-echo/src/main/java/sample/ws/echo/EchoSocket.java`

```java
package sample.ws.echo;

import javax.websocket.CloseReason;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;

//-> パスが "/echo" のリクエストを処理するクラス
@ServerEndpoint("/echo")
public class EchoSocket {
  //-> クライアントが接続してきたときの処理
  @OnOpen public void open(Session ssn) {
    System.out.println("@OnOpen: " + ssn.getId());
  }
  //-> クライアントがメッセージを送ってきたときの処理
  @OnMessage public String msg(String msg) {
    System.out.println("@OnMessage:");
    System.out.println(msg);
    return msg;
  }
  //-> クライアントが接続を閉じてきたときの処理
  @OnClose public void close(CloseReason rsn) {
    System.out.println("@OnClose: " + rsn);
  }
  //-> エラーが発生したときの処理
  @OnError public void error(Throwable t) {
    System.err.println("@OnError:");
    t.printStackTrace(System.err);
  }
}
```

エコーの処理は `@OnMessage` のメソッドで実装していて、クライアントのメッセージをそのまま返しています（`return msg;`）。


## 次回
次は、エコーアプリの画面（クライアントサイド）を、HTML や JavaScript を使って実装していこうと思います。

次の記事：[エコーアプリのUI開発](/entry/java/websocket/echo/dev-ui)
