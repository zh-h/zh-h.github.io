---
title: Kotlin objct 单例与反编译探究
date: 2017-10-25
tags: [Java,Kotlin]
categories: [Java]
---

对 Kotlin 单例模式的语法糖感兴趣，了解一下怎么实现对应的 Java 如何实现最优化。

Java 单例模式参考 [http://applehater.cn/2016/09/20/75/](常见 Java 单例模式的实现)

## object 对象表达式

使用 Kotlin 的 object 对象表达式可以很方便创建一个单例模式。
```
object cat {
    fun say() {
        println("meow~")
    }
}

fun main(args: Array<String>) {
    cat.say() // 直接使用对象名称访问
}
```

对象表达式在我们使用的地方立即初始化并执行的。

对象声明是懒加载的，是在我们第一次访问时初始化的。


### Java 实现

```
public final class cat {
   public static final cat INSTANCE;

   public final void say() {
      String var1 = "meow~";
      System.out.println(var1);
   }

   private cat() {
      INSTANCE = (cat)this;
   }

   static {
      new cat();
   }
}

public final class The_object_typeKt {
   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      cat.INSTANCE.say();
   }
}
```

可见对象表达式使用了静态代码块构造的单例模式，静态代码块是在类初始化的时候进行的，而不像静态成员变量在类装载的时候就行初始化，可以实现延迟加载；利用 JVM 静态代码初始化的特性，可以实现同步的单例模式。

## companion object 伴随对象

Kotlin 没有提供静态方法，使用伴随对象可以获得近似静态成员访问的效果，但是实际上仍然是真正的成员实例。

```
class AnimalProtectionOrganization{
    fun say(){
        println("protect!")
    }
}

class Dog{
    companion object {
        val APO = AnimalProtectionOrganization()
    }
}

fun main(args: Array<String>) {
    Dog.APO.say()
}
```
伴随对象在对应的类加载初始化，和静态变量的机制类似。

### Java 实现

```
public final class Dog {
   @NotNull
   private static final AnimalProtectionOrganization APO = new AnimalProtectionOrganization();
   public static final Dog.Companion Companion = new Dog.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      @NotNull
      public final AnimalProtectionOrganization getAPO() {
         return Dog.APO;
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}

public final class The_object_typeKt {
    public static final void main(@NotNull String[] args) {
        Intrinsics.checkParameterIsNotNull(args, "args");
        Dog.Companion.getAPO().say();
    }
}
```

使用了内部嵌套类实现单例模式。

## Kotlin 反编译为 Java 源码

1. 使用IntelliJ IDEA 打开制定的 Kotlin 文件(需安装Kotlin插件)；

2. 使用 Tools -> Kotlin -> Show Kotlin Bytecode；

    ![show kotlin bytecode](http://wx4.sinaimg.cn/large/e7c91439gy1filpinehauj20k80dsjsh.jpg)

3. 使用 decompile 将 JVM 字节码转换为对应的 Java 源码；

    ![decomplie bytecode to java](http://wx3.sinaimg.cn/large/e7c91439gy1filpimjiaoj20k80cfdgm.jpg)

4. 然后你就可以探究一下 Kotlin 的语法糖是怎么在 Java 中实现。