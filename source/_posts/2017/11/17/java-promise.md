---
title: Java Promise 实现
date: 2017-11-17
tags: [promise]
categories: [Java]
---

## 使用 ExcuteService

可以根据自己的需要来创建壹個 ExecutorService ，也可以使用 Executors 工厂方法来创建壹個 ExecutorService 实例。这里有几個创建 ExecutorService 的例子：
```java
    ExecutorService executorService1 = Executors.newSingleThreadExecutor();  
    ExecutorService executorService2 = Executors.newFixedThreadPool(10);  
    ExecutorService executorService3 = Executors.newScheduledThreadPool(10);  
```
### ExecutorService 使用方法

这里有几种不同的方式让你将任务委托给壹個 ExecutorService：
```java
execute(Runnable)  
submit(Runnable)  
submit(Callable)  
invokeAny(...)  
invokeAll(...)  
```
#### execute(Runnable)

方法 execute(Runnable) 接收壹個 java.lang.Runnable 对象作为参数，并且以异步的方式执行它。如下是壹個使用 ExecutorService 执行 Runnable 的例子：
```java
ExecutorService executorService = Executors.newSingleThreadExecutor();  
    
executorService.execute(new Runnable() {  
    public void run() {  
        System.out.println("Asynchronous task");  
    }  
});  
        
executorService.shutdown();  
```

使用这种方式没有办法获取执行 Runnable 之后的结果，如果你希望获取运行之后的返回值，就必须使用 接收 Callable 参数的 execute() 方法，后者将会在下文中提到。

#### submit(Runnable)

方法 submit(Runnable) 同样接收壹個 Runnable 的实现作为参数，但是会返回一个 Future 对象。这個 Future 对象可以用于判断 Runnable 是否结束执行。如下是壹個 ExecutorService 的 submit() 方法的例子：
```java
Future future = executorService.submit(new Runnable() {  
    public void run() {  
        System.out.println("Asynchronous task");  
    }  
});  
//如果任务结束执行则返回 null  
System.out.println("future.get()=" + future.get());  
```
#### submit(Callable)

方法 submit(Callable) 和方法 submit(Runnable) 比较类似，但是区别则在于它们接收不同的参数类型。Callable 的实例与 Runnable 的实例很类似，但是 Callable 的 call() 方法可以返回壹個结果。方法 Runnable.run() 则不能返回结果。

Callable 的返回值可以从方法 submit(Callable) 返回的 Future 对象中获取。如下是壹個 ExecutorService Callable 的样例：
```java
Future future = executorService.submit(new Callable(){  
    public Object call() throws Exception {  
        System.out.println("Asynchronous Callable");  
        return "Callable Result";  
    }  
});  
    
System.out.println("future.get() = " + future.get());  
```


上述样例代码会输出如下结果：
```java
ExecutorService executorService = Executors.newSingleThreadExecutor();  
    
Set<Callable<String>> callables = new HashSet<Callable<String>>();  
    
callables.add(new Callable<String>() {  
    public String call() throws Exception {  
        return "Task 1";  
    }  
});  
callables.add(new Callable<String>() {  
    public String call() throws Exception {  
        return "Task 2";  
    }  
});  
callables.add(new Callable<String>() {  
    public String call() throws Exception {  
        return "Task 3";  
    }  
});  
    
String result = executorService.invoke(callables);  
    
System.out.println("result = " + result);  
    
executorService.shutdown();
```

如果把当中 Callable 的方法作为 Http 请求，那么就可以实现多个 HTTP 请求并发，一个请求异常整个调用异常，最后一个请求完成，整个调用完成。极大利用了 Java 的线程管理和网络带宽。