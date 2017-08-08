---
title: Java 8 的一些使用技巧
date: 2017-10-04
tags: [Java]
categories: [Java]
---

Java 8 发布三年后，今年年底将要发布 Java 9。然而对于很多使用者来说，受限与旧项目的稳定性考虑以及框架封装隐藏底层 API，很多 Java 8 的特性都没有使用上。

刚刚受封为 Android 官方语言的 Kotlin 似乎热潮已经冷却，虽然 Kotlin 有着甜蜜蜜的语法糖，但是表面易用的语法实际上却埋下了很多坑（[解毒 Kotlin Koans: 02 震惊！你的 Java 代码居然被转换成了这样...](https://blog.kotliner.cn/2017/07/30/KotlinKoans-02-Java-Kotlin-Conversion/)），上线使用还要做很多深入学习。


为了迎接 Java 9，还是先把 Java 8 用熟。。。

## 文件操作

以往为了读取一个文本文件的内容可能需要需要十几行代码。

```java
public static String readTranslateJsonFileToString(String resourceName) {
    ClassLoader classLoader = LocationTranslateUtil.class.getClassLoader();
    URL resource = classLoader.getResource(resourceName);
    StringBuilder content = new StringBuilder();
    if (resource != null) {
        File file = new File(resource.getFile());
        InputStream is;
        try {
            is = new FileInputStream(file);
            InputStreamReader isr = new InputStreamReader(is, "UTF-8");
            BufferedReader br = new BufferedReader(isr);
            String temp;
            while ((temp = br.readLine()) != null) {
                content.append(temp);
            }
        } catch (IOException e) {
            // TODO
        }
    }
    return content.toString();
}
```

使用 `java.nio.*` 缩短为四行，读取类路径中的文件。
```java
public String readPropertiesStr() throws IOException, URISyntaxException{
    ClassLoader classLoader = this.getClass().getClassLoader();
    URL resource = classLoader.getResource("config.properties");
    Path path = Paths.get(resource.toURI());
    byte[] data = Files.readAllBytes(path);
    return new String(data, StandardCharsets.UTF_8);
}
```

## 时间日期

### 时间格式化

```java
DateTimeFormatter df = DateTimeFormatter.ISO_LOCAL_DATE_TIME; // 线程安全

public String getDateTimeString() {
    LocalDateTime now = LocalDateTime.now();
    return df.format(now);
}

public LocalDateTime parseToLocalDateTime(){
    return LocalDateTime.parse("2017-08-14T12:49:44.133", df);
}
```

新增的 `DateTimeFormatter` 是不可变，而且是线程安全的类，因此可以直接使用在成员变量中。

### 明天，去年

```java
public LocalDateTime nextDate(){
    return LocalDateTime.now().plusDays(1);
}

public LocalDateTime previousYear(){
    return LocalDateTime.now().minusYears(1);
}
```

### 时间差

快速计算两个本地时间的时间差，包括`NANOS, MICROS, MILLIS, SECONDS, MINUTES, HOURS, HALF_DAYS, DAYS, WEEKS, MONTHS, YEARS, DECADES, CENTURIES, MILLENNIA, ERAS` 时间单位

```
public long differDates() {
    for(ChronoUnit unit : ChronoUnit.values()){
        System.out.println(unit.name());
    }
    return ChronoUnit.DAYS.between(previousYear(),LocalDateTime.now());
}
```

### 早晚

比较两个日期/时间的早晚，可以使用`isAfter`或者`isBefore` 方法
```
LocalDate tomorrow = LocalDate.of(2017, 8, 15);
if(tommorow.isAfter(LocalDate.now())){
    System.out.println("Tomorrow comes after today");
}
```

### 时区

```
public LocalDateTime utcNow(){
    return LocalDateTime.now(ZoneId.of("Europe/Berlin"));
}
```

### 转换

```
public Date toOldStyleDate(){
    Instant instance = LocalDateTime.now().atZone(ZoneId.of("Asia/Shanghai")).toInstant();
    return Date.from(instance);
}
```
## 默认方法的接口

Java 8 允许我们使用default关键字，为接口声明添加非抽象的方法实现。这个特性又被称为扩展方法。

### `defaut`关键字

接口也可以有多个方法实现，使用`defaut`关键字修饰。

```java
interface USB {
    String ping();

    default int getPinNum() {
        return 4;
    }
}

public class DefaultMethodInInterface {
    public static void main(String[] args) {
        USB usb2_0 = new USB() {
            @Override
            public String ping() {
                return "OK";
            }
        };

        System.out.println(usb2_0.getPinNum());
    }
}
```

### Java 多继承

既然接口都可以包含多个方法实现，那么Java本身可以实现多个接口，这样是不是可以实现多继承？

可以实现Java的多继承。

```java
interface USB {
    String ping();

    default int getPinNum() {
        return 4;
    }
}

interface Plugin {
    default String plug() {
        return "biu";
    }
}

class USB3_0 implements USB, Plugin {

    @Override
    public String ping() {
        return "OK";
    }
}
public class DefaultMethodInInterface {
    public static void main(String[] args) {
        USB3_0 usb3_0 = new USB3_0();
        System.out.println(usb3_0.getPinNum());
        System.out.println(usb3_0.plug());
    }
}
```
## Lambda

省略匿名类实现单个方法的接口，直接使用单行表达式或者块。

```java
List<String> names = Arrays.asList("hua", "zohar", "zonghua");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) { // 前者比后者小
        System.out.println(o1 + " --- " + o2);
        return (int) o1.toCharArray()[0] - (int) o2.toCharArray()[0];
    }
});

// 方法引用 String::compareTo
Collections.sort(names, (o1, o2) -> o1.compareTo(o2));
```

## 函数式接口

### @FunctionalInterface 注解

使用该注解声明一个接口只包含一个抽象方法，但是可以包含多个默认方法实现。

```java
@FunctionalInterface
interface SomeThing{
    String say();
}

public class TheFunctionalInterface {

    public static void main(String[] args) {
        SomeThing someThing = () -> "lala";
        System.out.println(someThing.say());
    }
}
```

### 方法引用

可以从类或者实例中获取方法引用，使用`::`符号

```java
class SomeClass {
    String say() {
        return "haha";
    }
}

class OtherClass {
    static String say(){
        return "66666";
    }
}
public class TheMethodReference {
    public static void main(String[] args) {
        SomeClass someObject = new SomeClass();
        SomeThing someThing = someObject::say;
        SomeThing otherThing = OtherClass::say;
        System.out.println(someThing.say());
        System.out.println(otherThing.say());
    }
}
```

### 作用域

与匿名类的访问类似，实际上访问外部变量只能是`final`修饰变量，虽然可以省略，但再次变更变量会导致编译出错。

```java
@FunctionalInterface
interface Lala {
    String say();
}

public class TheLambdaContext {
    public static void main(String[] args) {
        String[] name = new String[]{"Lily"};
        Lala lala = () -> name[0] + " hehe";
        System.out.println(lala.say());
//        name = new String[]{"What"}; 不可变更引用或者值
        lala = () -> {
            name[0] = "Zoar";
            return name[0];
        };
        System.out.println(lala.say());
    }
}
```

## 内嵌函数式接口

### Functions

接收一个传入参数，返回结果，使用默认的方法。

```java
Function<String, Integer> lala = (str) -> 1; // <input,return>
System.out.println(lala.apply("lala").addThen(Objects::notNull));

Supplier<String> hehe = () -> "lala"; // 没有输入参数
System.out.println(hehe.get());

Consumer<String> whatsUp = (str) -> str = str.substring(0, 1);
String someThing = "lala";
whatsUp.accept(someThing);
System.out.println(someThing);

Comparator<String> strCompare = (o1, o2) -> o1.compareTo(o2); // 前者比后者大为正
System.out.println(strCompare.compare("a", "b"));
```

### Predicates

传入参数，返回布尔值。具有逻辑方法`or` `and` `negate`。

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

### Suppliers

没有传入参数，返回给定类型。

```java
Supplier<String> hehe = () -> "lala"; // 没有输入参数
System.out.println(hehe.get());
```

### Consumers

没有返回参数，给定输入类型。

```java
Consumer<String> whatsUp = (str) -> str = str.substring(0, 1);
String someThing = "lala";
whatsUp.accept(someThing);
System.out.println(someThing);
```


### Comparators

对比两个传入参数，左边的比右边的大就返回正值，反则负值，相等为0。

```java
Comparator<String> strCompare = (o1, o2) -> o1.compareTo(o2); // 前者比后者大为正
System.out.println(strCompare.compare("a", "b"));
```

### Optionals

一个存储结果的容器，用来省掉函数过程中的`if xx != null`。

```java
Optional<String> optional = Optional.of("lala");

optional.isPresent();           // true
optional.get();                 // "lala"
optional.orElse("hehe");    // "lala"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "l"
```

## 流处理

### Filter

过滤掉集合中的某些元素，传入参数是元素，返回过滤判断的布尔值。

```java
names.stream()
        .filter(Objects::nonNull)
        .map(String::toUpperCase)
        .sorted(((o1, o2) -> -o1.compareTo(o2)))
        .forEach(System.out::println);
```

### Sorted

传入前后两个参数，返回两者判断的有符号整数。

```java
names.stream()
        .filter(Objects::nonNull)
        .map(String::toUpperCase)
        .sorted(((o1, o2) -> -o1.compareTo(o2)))
        .forEach(System.out::println);
```

**原来集合的顺序并不会被改变**

### Map

传入一个元素，处理后返回这个元素。

```java
names.stream()
        .filter(Objects::nonNull)
        .map(String::toUpperCase)
        .sorted(((o1, o2) -> -o1.compareTo(o2)))
        .forEach(System.out::println);
```

### anyMatch

在集合中查找匹配的元素。

```java
boolean hasStartWith0 = names.stream()
                .anyMatch((s) -> s.startsWith("0"));
        System.out.println(hasStartWith0);
```

### Reduce

传入最终元素、每个元素，把每个元素处理到最终的元素中。

```java
Optional<String> namesStr = names.stream()
        .filter(TheStreams::nonNull)
        .reduce((o1, o2) -> {
            System.out.println(o2);
            return o1 + " - " + o2;
        }); // 最终元素,每个元素
namesStr.ifPresent(System.out::println);
```

### Count

计算集合中的数量

```java
long count = names.stream()
        .filter(TheStreams::nonNull)
        .filter((s) -> s.startsWith("0") || s.startsWith("L"))
        .count();
System.out.println(count);
```

### 并行化

Java 8 流操作内部封装了多线程调用，可以使用流操作进行并行处理。

```java
public static List<String> generateRandomNameList() {
    int max = 1000000;
    List<String> values = new ArrayList<>(max);
    for (int i = 0; i < max; i++) {
        UUID uuid = UUID.randomUUID();
        values.add(uuid.toString());
    }
    return values;
}

public static long sequenceSort(List<String> names) {
    return names.stream()
            .sorted()
            .count(); // 3s518ms
}

public static long parallelSort(List<String> names) {
    return names.parallelStream()
            .sorted()
            .count(); //2s134ms
}
```
