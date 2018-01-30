# Java String Constant Pool (Java字符串常量池)

当你在Java中声明一个新的字符串时，在这个场景下有一些有趣的事情发生。

这是一个基本的字符串声明，我们创建了一个新的字符串变量`employee`并给它赋值。

```java
String employee = "Edgar Allen Poe";
```

Java不仅会创建变量`employee`，而且还会为内存中的字面值“Edgar Allen Poe”分配空间。内存中的这个区域被称为**字符串常量池**。它就像程序的其他部分可用的字符串值池。

## 重用字符串常量池中的值

现在，如果你创建了另一个变量，比如`employee2`，并且还给了它一个“Edgar Allen Poe”的值，那么Java只是重用了已经在池中的值。

```java
String employee2 = "Edgar Allen Poe";
```

![](https://study.com/cimages/multimages/16/9796710e-4789-4f7b-8140-c7a179bc4fd4_java_constant_pool_literals.png)

你会注意到字符串常量池位于内存的**堆**部分。

## 创建一个新的字符串实例

如果你创建String类的新实例，则常量池的工作方式不同。让我们创建另一个变量`employee3`，并给它相同的字面值。但是，这次我们将创建一个String类的新实例：

```java
String employee3 = new String("Edgar Allen Poe");
```

当这个代码被处理时，Java将会有所不同。而不是再次使用相同的字面值，它会在内存中创建一个新的值。在这种情况下，它不会在字符串常量池中创建它，而是在内存堆中创建它。

![](https://study.com/cimages/multimages/16/1aecb034-2dd9-4ae6-8909-48de011292ed_java_constant_pool_new_objects.png)


