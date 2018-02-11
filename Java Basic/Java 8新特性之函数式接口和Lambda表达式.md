# Java 8新特性之函数式接口和Lambda表达式

面向对象并不坏，但它给程序带来了很多冗长的内容。例如，假设我们要创建一个Runnable的实例。通常我们使用下面的匿名类来完成它：

```java
Runnable r = new Runnable() {
        @Override
        public void run() {
            System.out.println("My Runnable");
        }
    };
```
如果你看看上面的代码，实际使用的部分是 run() 方法中的代码。其余所有的代码是因为Java程序的结构化方式。

Java 8函数式接口和Lambda表达式通过删除大量的样板代码，帮助我们编写更少、更简洁的代码。

## 函数式接口

具有一个抽象方法的接口称为函数式接口。添加了`@FunctionalInterface`注解，以便我们可以将接口标记为函数式接口。

使用它不是强制性的，但最好的做法是将它与函数式接口一起使用，以避免意外添加额外的方法。如果接口使用`@FunctionalInterface`注解进行注释，并且我们尝试使用多个抽象方法，则会引发编译器错误。

**Java 8函数式接口的主要好处是我们可以使用lambda表达式来实例化它们，避免使用笨重的匿名类实现。**

Java 8 Collections API已被重写，并引入了新的Stream API，它使用了许多函数式接口。 Java 8在`java.util.function`包中定义了很多函数式接口。一些有用的java 8函数式接口如`Consumer`、`Supplier`、`Function`和`Predicate`。

`java.lang.Runnable`是使用单一抽象方法 run() 的函数式接口的一个很好的例子。

### 下面的代码片段为函数式接口提供了一些指导：

```java
interface Foo { boolean equals(Object obj); }
// Not functional because equals is already an implicit member (Object class)

interface Comparator<T> {
 boolean equals(Object obj);
 int compare(T o1, T o2);
}
// Functional because Comparator has only one abstract non-Object method

interface Foo {
  int m();
  Object clone();
}
// Not functional because method Object.clone is not public

interface X { int m(Iterable<String> arg); }
interface Y { int m(Iterable<String> arg); }
interface Z extends X, Y {}
// Functional: two methods, but they have the same signature

interface X { Iterable m(Iterable<String> arg); }
interface Y { Iterable<String> m(Iterable arg); }
interface Z extends X, Y {}
// Functional: Y.m is a subsignature & return-type-substitutable

interface X { int m(Iterable<String> arg); }
interface Y { int m(Iterable<Integer> arg); }
interface Z extends X, Y {}
// Not functional: No method has a subsignature of all abstract methods

interface X { int m(Iterable<String> arg, Class c); }
interface Y { int m(Iterable arg, Class<?> c); }
interface Z extends X, Y {}
// Not functional: No method has a subsignature of all abstract methods

interface X { long m(); }
interface Y { int m(); }
interface Z extends X, Y {}
// Compiler error: no method is return type substitutable

interface Foo<T> { void m(T arg); }
interface Bar<T> { void m(T arg); }
interface FooBar<X, Y> extends Foo<X>, Bar<Y> {}
// Compiler error: different signatures, same erasure
```

## Lambda表达式

由于在函数式接口中只有一个抽象函数，因此在将该lambda表达式应用于该方法时不会出现混淆。 Lambda表达式的语法是 **(argument) -> (body)**。现在来看看如何使用lambda表达式写上面的匿名Runnable:

```java
Runnable r1 = () -> System.out.println("My Runnable");
```
让我们试着了解上面的lambda表达式中发生了什么。

* Runnable是一个函数式接口，这就是为什么我们可以使用lambda表达式来创建它的实例。 
* 由于 run() 方法没有参数，我们的lambda表达式也没有参数。 
* 就像if-else块一样，我们可以避免大括号({})，因为我们在方法体中只有单个语句。对于多个语句，我们必须像使用其他方法一样使用花括号。

## 为什么我们需要Lambda表达式

#### 1、减少代码行数

使用lambda表达式的一个明显优势是代码量减少了，我们已经看到，我们可以轻松地使用lambda表达式而不是使用匿名类来创建函数接口的实例。

#### 2、顺序和并行执行支持

使用lambda表达式的另一个好处是我们可以从Stream API顺序和并行操作支持中受益。

为了解释这一点，我们举一个简单的例子，我们需要编写一个方法来测试一个数字是否是素数。

```java
//Traditional approach
private static boolean isPrime(int number) {		
	if(number < 2) return false;
	for(int i=2; i<number; i++){
		if(number % i == 0) return false;
	}
	return true;
}
```
上述代码的问题在于它本质上是顺序的，如果数字非常大，那么它将花费大量时间。代码的另一个问题是有太多的退出点使其可读性差。

```java
//Declarative approach
private static boolean isPrime(int number) {		
	return number > 1
			&& IntStream.range(2, number).noneMatch(
					index -> number % index == 0);
}
```
为了提高可读性，我们也可以编写如下的方法。

```java
private static boolean isPrime(int number) {
	IntPredicate isDivisible = index -> number % index == 0;
	
	return number > 1
			&& IntStream.range(2, number).noneMatch(
					isDivisible);
}
```

#### 3、将行为传递给方法

我们来看看如何使用lambda表达式来传递一个方法的行为。假设我们必须编写一个方法来对列表中的数字求和，如果它们符合给定的条件。我们可以使用Predicate并编写如下的方法。

```java
public static int sumWithCondition(List<Integer> numbers, Predicate<Integer> predicate) {
	    return numbers.parallelStream()
	    		.filter(predicate)
	    		.mapToInt(i -> i)
	    		.sum();
	}
```
使用示例：

```java
//sum of all numbers
sumWithCondition(numbers, n -> true)
//sum of all even numbers
sumWithCondition(numbers, i -> i%2==0)
//sum of all numbers greater than 5
sumWithCondition(numbers, i -> i>5)
```
#### 4、使用惰性求值效率更高

使用lambda表达式的另一个优点是惰性求值(lazy evaluation)。假设我们需要编写一个方法来找出3到11范围内的最大奇数并返回它的平方。

通常我们会为此方法编写如下代码：

```java
private static int findSquareOfMaxOdd(List<Integer> numbers) {
		int max = 0;
		for (int i : numbers) {
			if (i % 2 != 0 && i > 3 && i < 11 && i > max) {
				max = i;
			}
		}
		return max * max;
	}
```

上面的程序总是按顺序运行，但我们可以使用Stream API来实现这一点，并获得Laziness-seeking的好处。我们来看看如何使用Stream API和lambda表达式以函数式编程方式重写此代码。

```java
public static int findSquareOfMaxOdd(List<Integer> numbers) {
		return numbers.stream()
				.filter(NumberTest::isOdd) 		     //Predicate is functional interface and
				.filter(NumberTest::isGreaterThan3)	// we are using lambdas to initialize it
				.filter(NumberTest::isLessThan11)	// rather than anonymous inner classes
				.max(Comparator.naturalOrder())
				.map(i -> i * i)
				.get();
	}

	public static boolean isOdd(int i) {
		return i % 2 != 0;
	}
	
	public static boolean isGreaterThan3(int i){
		return i > 3;
	}
	
	public static boolean isLessThan11(int i){
		return i < 11;
	}
```
















