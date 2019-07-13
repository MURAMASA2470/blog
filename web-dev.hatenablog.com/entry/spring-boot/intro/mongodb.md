---
Title: SpringBoot：MongoDBにアクセス
Category:
- Java
Date: 2017-02-26T16:48:00+09:00
URL: https://web-dev.hatenablog.com/entry/spring-boot/intro/mongodb
EditURL: https://blog.hatena.ne.jp/mamorums/web-dev.hatenablog.com/atom/entry/10328749687179109191
---

SpringBoot の Webアプリから、MongoDB にアクセスする方法を書きます。データ（≒ドキュメント, JSON）を登録したり、検索したりしてみます。

## 環境・ツール
- JDK 1.8 以上
- Maven 3.0 以上（or Gradle 等）
- MongoDB（執筆時 3.4.2）


## 手順1. ビルドファイルの作成
Maven のビルドファイルを作成します。アプリのルートディレクトリは `gssb-nosql` としています。

`gssb-nosql/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.github.mamorum</groupId>
  <artifactId>gssb-nosql</artifactId>
  <version>1.0.0</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
  </dependencies>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

MongoDB 用の依存性 `spring-boot-starter-data-mongodb` を追加しています。


## 手順2. ドメイン（エンティティ）の作成
データを保持するクラスを作成します。

`gssb-nosql/src/main/java/gssb/nosql/mongodb/model/Customer.java`

```java
package gssb.nosql.mongodb.model;

import org.springframework.data.annotation.Id;

public class Customer {
  @Id public String id;
  public String firstName, lastName;
}
```

## 手順3. リポジトリの作成
データを操作するインターフェイスを作成します。

`gssb-nosql/src/main/java/gssb/nosql/mongodb/repository/CustomerRepository.java`

```java
package gssb.nosql.mongodb.repository;

import org.springframework.data.mongodb.repository.MongoRepository;

import gssb.nosql.mongodb.model.Customer;

public interface CustomerRepository extends MongoRepository<Customer, String> {
  Iterable<Customer> findByLastName(String lastName);
}
```

MongoRepository を継承すると、基本的なデータ操作（find, save, etc）用のメソッドが追加されます。また、規約に従ってメソッドを実装すると、条件などを指定できます。

詳細は、[Spring Data MongoDB のドキュメント](http://docs.spring.io/spring-data/data-mongo/docs/current/reference/html/) に書かれています。


## 手順4. コントローラの作成
リクエストを受け付けて、リポジトリに処理を移譲するクラスを作成します。

`gssb-nosql/src/main/java/gssb/nosql/mongodb/controller/CustomerController.java`

```java
package gssb.nosql.mongodb.controller;

import java.util.Collections;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import gssb.nosql.mongodb.model.Customer;
import gssb.nosql.mongodb.repository.CustomerRepository;

@RestController @RequestMapping(path="/customers")
public class CustomerController {

  @Autowired CustomerRepository repo;
  
  @RequestMapping(method=RequestMethod.POST)
  public Map<String, String> create(
    @RequestBody Customer customer
  ) {
    Customer created = repo.save(customer);
    return Collections.singletonMap("id", created.id);
  }

  @RequestMapping(method=RequestMethod.GET)
  public Map<String, Iterable<Customer>> find(
    @RequestParam String lastName
  ) {
    Iterable<Customer> finded = repo.findByLastName(lastName);
    return Collections.singletonMap("customers", finded);
  }
}
```

リクエストボディ（JSON）の Customer を登録するメソッドと、リクエストパラメータの lastName で検索するメソッドを実装しています。


## 手順5. 起動クラスの作成
SpringBoot のアプリを起動するクラスを作成します。

`gssb-nosql/src/main/java/gssb/nosql/App.java`

```java
package gssb.nosql;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
  public static void main(String[] args) {
    SpringApplication.run(App.class);
  }
}
```


## 手順6. 起動
次のコマンドでアプリを起動します。（事前に MongoDB をローカルで起動している想定です。）


```
gssb-nosql > mvn spring-boot:run
```


## 手順7. 確認
動作確認には curl を使います。次のコマンドで、データを１つ登録してみます（コントローラの create メソッド実行）。

`実行コマンド（※ JSON 内のエスケープ文字「\」は Windows で必要）`

```
curl -H "Content-Type: application/json" -d "{\"firstName\":\"Taro\", \"lastName\":\"Suzuki\"}" http://localhost:8080/customers
```

`実行結果`

```
{"id":"575397dcb465790812d7d99c"}
```

次は、登録したデータを検索してみます（コントローラの find メソッド）。

`実行コマンド`

```
curl http://localhost:8080/customers?lastName=Suzuki
```

`実行結果`

```
{"customers":[{"id":"575397dcb465790812d7d99c","firstName":"Taro","lastName":"Suzuki"}]}
```

## 補足1. 接続先の設定
SpringBoot は、デフォルトだと MongoDB（ローカル）のデータベース `test` に接続するようです。接続先を変更したい場合は、`application.properties` などで設定します。


## 補足2. MongoDB のデータ
MongoDB のシェルでデータを確認すると、`"_class"` というプロパティが追加されているようでした。

`実行コマンド（MongoDB shell）`

```
> db.customer.find()
```

`実行結果`

```
{ "_id" : ObjectId("575397dcb465790812d7d99c"), "_class" : "gssb.nosql.mongodb.model.Customer", "firstName" : "Taro", "lastName" : "Suzuki" }
```

Spring の仕様なのかと思います。


## ソースコード
[gssb-nosql - GitHub](https://github.com/mamorum/blog/tree/master/code/gssb-nosql)  
※ プロジェクト名の gssb は、Getting Started Spring Boot の略です。
