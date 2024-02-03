---
layout: post
title: "jvm调优机制"
date:   2019-11-22
tags: [geek]
comments: true
author: lemonchann
toc: trueh
---



### 

当我们要申请一个资源时，我们为了确保它能被关闭，即使遇见了异常，也必须要执行finally语句中的关闭资源部分。

```java
BufferedWriter writer = null;
try {
    writer = new BufferedWriter(new FileWriter(fileName));
    writer.write(str);  // do something with the file we've opened
} catch (IOException e) {
   // handle the exception
} finally {
    try {
        if (writer != null)
            writer.close();
    } catch (IOException e) {
       // handle the exception
    }
}
```



如果我们使用try-with-resources语句，就不需要使用finally回收资源了

```java
try(BufferedWriter writer = new BufferedWriter(new FileWriter(fileName))){
    writer.write(str); // do something with the file we've opened
}
catch(IOException e){
    // handle the exception
}
```



当我们需要声明复数个要回收的类时

```java
try (BufferedWriter writer = new BufferedWriter(new FileWriter(fileName));
    Scanner scanner = new Scanner(System.in)) {
if (scanner.hasNextLine())
    writer.write(scanner.nextLine());
}
catch(IOException e) {
    // handle the exception
}
```

从Java 9开始，没有必要在*try-with-resources*语句中声明*资源*。我们可以这样做：

```java
BufferedWriter writer = new BufferedWriter(new FileWriter(fileName));
try (writer) {
    writer.write(str); // do something with the file we've opened
}
catch(IOException e) {
    // handle the exception
}
```



### 支持的类

声明的所有资源`try()`必须实现该`AutoCloseable`接口。这些通常是各种类型的编写器，读取器，套接字，输出或输入流等`resource.close()`。使用完所有需要编写的内容。

当然，这包括实现该`AutoClosable`接口的用户定义对象。但是，您很少会遇到想要编写自己的资源的情况。

万一发生这种情况，您需要实现`AutoCloseable`或`Closeable`（仅在此处保持向后兼容性，最好使用`AutoCloseable`）接口并重写该`.close()`方法：

订阅我们的新闻
在收件箱中获取临时教程，指南和作业。从来没有垃圾邮件。随时退订。

订阅电子报
订阅

```text
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        // close your resource in the appropriate way
    }
}
```
