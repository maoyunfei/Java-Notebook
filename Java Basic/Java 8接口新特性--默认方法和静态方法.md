# Java 8接口新特性之默认方法和静态方法

Java 8接口新特性包括接口中的静态方法和默认方法。在Java 8之前，我们只能在接口中使用方法声明。但是从Java 8开始，我们可以在接口中使用默认方法和静态方法。

## 默认方法

为了在java接口中创建一个默认方法，我们需要在方法签名中使用“**default**”关键字。例如，

```java
public interface Interface1 {

	void method1(String str);
	
	default void log(String str){
		System.out.println("I1 logging::"+str);
	}
}
```
注意到 log(String str) 是`Interface1`中的默认方法。现在当一个类实现`Interface1`时，并不强制为接口的默认方法提供实现。这个特性将帮助我们扩展接口和其他方法，我们所需要的只是提供一个默认实现。

另一个接口定义如下：

```java
public interface Interface2 {

	void method2();
	
	default void log(String str){
		System.out.println("I2 logging::"+str);
	}

}
```

**我们知道Java不允许我们继承多个类，因为它会导致“钻石问题”，因为编译器无法决定使用哪个超类方法。使用默认方法，"钻石问题"也会出现在接口上。因为如果一个类同时实现了`Interface1`和`Interface2`并且没有实现通用的默认方法，编译器无法决定选择哪一个。**(备注：[The diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem))

实现多个接口是Java不可或缺的组成部分，可以在核心Java类以及大多数企业应用程序和框架中找到它。所以为了确保这个问题不会发生在接口中，必须为通用的接口默认方法提供实现。因此，如果一个类正在实现上述两个接口，它将不得不为 log() 方法提供实现，否则编译器将抛出编译时错误。

一个同时实现`Interface1`和`Interface2`的简单类如下：

```java
public class MyClass implements Interface1, Interface2 {

	@Override
	public void method2() {
	}

	@Override
	public void method1(String str) {
	}

	@Override
	public void log(String str){
		System.out.println("MyClass logging::"+str);
		Interface1.print("abc");
	}
}
```

### 有关java接口默认方法的重点：

* Java 8接口的默认方法将帮助我们扩展接口，而不用担心会破坏实现类。 
* Java 8接口默认方法弥合了接口和抽象类之间的差异。 
* Java 8接口的默认方法将帮助我们避免utility类，比如所有的Collections类方法都可以在接口本身中提供。 
* Java接口的默认方法将帮助我们去除基础实现类，我们可以提供默认的实现，实现类可以选择重写哪一个。 
* 在接口中引入默认方法的主要原因之一是增强Java 8中的Collections API以支持lambda表达式。 
* 如果层次结构中的任何类具有相同签名的方法，则默认方法变得不相关。默认方法不能从`java.lang.Object`中覆盖方法。推理非常简单，这是因为Object是所有java类的基类。所以即使我们把Object类的方法定义为接口中的默认方法，也是无用的，因为总是使用Object类的方法。这就是为什么要避免混淆，我们不能有覆盖Object类方法的默认方法。 
* Java接口默认方法也被称为Defender方法或虚拟扩展方法。

## 静态方法

接口静态方法和默认方法类似，除了我们不能在实现类中override它们。这个特性有助于我们避免在实现类中由于不好的实现导致的不当结果。

```java
public interface MyData {

	default void print(String str) {
		if (!isNull(str))
			System.out.println("MyData Print::" + str);
	}

	static boolean isNull(String str) {
		System.out.println("Interface Null Check");

		return str == null ? true : "".equals(str) ? true : false;
	}
}
```
现在来看一个实现类，该类具有 isNull() 方法，但实现效果较差。

```java
public class MyDataImpl implements MyData {

	public boolean isNull(String str) {
		System.out.println("Impl Null Check");

		return str == null ? true : false;
	}
	
	public static void main(String args[]){
		MyDataImpl obj = new MyDataImpl();
		obj.print("");
		obj.isNull("abc");
	}
}
```
注意 isNull(String str) 是一种简单的类方法，它不会覆盖接口方法。例如，如果我们将`@Override`注释添加到 isNull() 方法，则会导致编译器错误。

现在，当我们运行应用程序时，我们得到以下输出:

```java
Interface Null Check
Impl Null Check
```
如果我们将接口方法从静态变为默认，我们将得到以下输出:

```java
Impl Null Check
MyData Print::
Impl Null Check
```

Java接口静态方法仅对接口方法可见，如果我们从 MyDataImpl 类中移除 isNull() 方法，我们将无法将其用于 MyDataImpl 对象。像其他静态方法一样，我们可以使用类名调用接口静态方法。例如：

```java
boolean result = MyData.isNull("abc");
```

### 有关java接口静态方法的重点：

* Java接口的静态方法是接口的一部分，我们不能用它来实现类对象。 
* Java接口静态方法适用于提供utility方法，例如空检查，集合排序等。 
* Java接口静态方法通过不允许实现类override它们来帮助我们提供安全性。 
* 我们不能为Object类方法定义接口静态方法，因为“这个静态方法不能从Object隐藏实例方法”，我们会得到编译器错误。这是因为它在java中是不允许的，因为Object是所有类的基类，我们不能有一个类级静态方法和另一个具有相同签名的实例方法。 
* 我们可以使用java接口的静态方法来移除Collections等utility类，并将它的所有静态方法移到相应的接口中，这很容易找到和使用。











