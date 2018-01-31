# equals()和hashCode()

[原文链接](https://dzone.com/articles/working-with-hashcode-and-equals-in-java)

默认情况下，Java超类`java.lang.Object`提供了两种比较对象的重要方法：`equals()`和`hashCode()`。在大型项目中实现多个类之间的交互时，这些方法变得非常有用。在本文中，我们将讨论这些方法之间的关系，它们的默认实现以及强制开发人员为每个方法提供自定义实现的情况。

## 方法定义和默认实现

```java
public class Object {
	...
	public boolean equals(Object obj) {
        return (this == obj);
    }
	...
	public native int hashCode();
	...
}
```
* `equal()`方法：JDK提供的默认实现是基于内存位置的 - **当且仅当它们存储在同一个内存地址中时，两个对象是相等的。**
* `hashCode()`方法：默认实现是个**本地方法**。需要满足三个约定：(1) 无论什么时间，在**同一个应用内**执行多次应该返回相同的值。(2) 如果两个对象的`equals()`方法返回`true`，那么它们的`hashCode()`必须相同。(3) 如果两个对象的`equals()`方法返回`false`，那么它们的`hashCode()`可以相同，也可以不同。

## 重写equals()

实例：

```java
package com.programmer.gate.beans;

public class Student {
    private int id;
    private String name;
    public Student(int id, String name) {
        this.name = name;
        this.id = id;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```


```java
@Override
public boolean equals(Object obj) {
    if (obj == null) return false;
    if (!(obj instanceof Student))
        return false;
    if (obj == this)
        return true;
    return this.getId() == ((Student) obj).getId();
}
```
### equals() With ArrayList

```java
public class HashcodeEquals {
    public static void main(String[] args) {
        Student alex = new Student(1, "Alex");
        List < Student > studentsLst = new ArrayList < Student > ();
        studentsLst.add(alex);
        System.out.println("Arraylist size = " + studentsLst.size());
        System.out.println("Arraylist contains Alex = " + studentsLst.contains(new Student(1, "Alex")));
    }
}
```
以上代码输出为：

```java
Arraylist size = 1
Arraylist contains Alex = true
```
**原因是`ArrayList`的`contains()`方法内部是调用的对象的`equals()`方法。**

## 重写hashCode()

```java
@Override
public int hashCode() {
    return id;
}
```

### equals() With HashSet

```java
public class HashcodeEquals {
    public static void main(String[] args) {
        Student alex1 = new Student(1, "Alex");
        Student alex2 = new Student(1, "Alex");
        HashSet < Student > students = new HashSet < Student > ();
        students.add(alex1);
        students.add(alex2);
        System.out.println("HashSet size = " + students.size());
        System.out.println("HashSet contains Alex = " + students.contains(new Student(1, "Alex")));
    }
}
```
以上代码输出为：

```java
HashSet size = 1
HashSet contains Alex = true
```
`HashSet`将其元素存储在内存桶中。每个桶都链接到一个特定的哈希码。当调用`students.add(alex1)`时，Java在存储桶中存储`alex1`并将其链接到`alex1.hashCode()`的值。现在任何时候，一个具有相同散列码的元素被插入到集合中，它将会替换掉`alex1`。但是，由于`alex2`具有不同的散列码，它将被存储在一个单独的存储桶中，并被视为完全不同的对象。

现在，当`HashSet`在其中搜索一个元素时，它首先生成元素的哈希码并查找与这个哈希码对应的一个桶。

**这同样适用于`HashMap`，`HashTable`或任何使用散列机制来存储元素的数据结构。`hashCode()`用于散列到桶，`equals()`用于判断对象是否相同。** 

