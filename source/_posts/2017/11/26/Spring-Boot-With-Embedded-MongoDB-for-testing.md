---
title: Spring Boot 使用内嵌 Mongo 进行测试
date: 2017-11-26
tags: [Spring Boot]
categories: [Java]
---

## 如何进行数据库操作层的单元测试

如何进行测试保证数据库操作层的语法正确，如果使用外部链接的数据库，不仅速度慢，数据定义麻烦而且违反了单元测试无外部依赖的规范。因此需要指定模拟的类库进行数据库操作，并且这个数据库是可以对语法进行检查。

## Embedded MongoDB

### 引入依赖

build.gradle
```groovy
dependencies {
    compile "org.mongodb:mongo-java-driver:2.12.2"


    testCompile "junit:junit:4.11"
    testCompile "de.flapdoodle.embed:de.flapdoodle.embed.mongo:1.46.0"
}
```

### 脚手架MongodbBaseTest

编写一个MongoBaseTest，这样所有需要Mongo的测试，可以继承这个类，就可以获取db了。

```java
public class MongodbBaseTest {
    private static final MongodStarter starter = MongodStarter.getDefaultInstance();
    protected MongoClient mongo;
    protected DB db;
    private MongodExecutable mongodExecutable;
    private MongodProcess mongod;

    @Before
    public void setUp() throws Exception {
        mongodExecutable = starter.prepare(new MongodConfigBuilder()
                .version(Version.Main.PRODUCTION)
                .net(new Net(12345, Network.localhostIsIPv6())).build());
        mongod = mongodExecutable.start();


        mongo = new MongoClient("localhost", 12345);
        db = mongo.getDB("embedded-mongo");
    }

    @After
    public void tearDown() throws Exception {
        mongod.stop();
        mongodExecutable.stop();
    }
}
```

### 编写UserTest
```java
public class UserTest extends MongodbBaseTest {
    private DBCollection users;

    @Override
    @Before
    public void setUp() throws Exception {
        super.setUp();
        users = db.getCollection("users");
    }

    @Test
    public void should_insert_and_get_user() {
        final DBObject userDocument = new BasicDBObjectBuilder()
                .add("name", "kiwi")
                .get();
        users.insert(userDocument);

        final DBObject userDocumentFromDb = users.findOne(new BasicDBObject("_id", userDocument.get("_id")));

        assertThat(userDocumentFromDb.get("name"), is("kiwi"));
    }
}
```

自此单元测试中有关数据库层都是 Embedded MongoDB 进行模拟，这样保证语法验证的同时避免了外部资源的依赖。

## Embedded MongoDB 与 Spring Test 结合