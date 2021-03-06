---
title: Java泛型
date: 2016-06-11 19:06:12
categories: Java
tags: 泛型
---




# 为什么需要泛型

<!--more-->


```java
Java

List list=new ArrayList();
list.add(1);
list.add(2);
list.add("3");
for(Object o:list){
	Integer i= (Integer) o;
	System.out.println(i);
}
```

以上代码先定义了一个List类型的集合，先向其中加入了两个Integer类型的值，随后加入一个String类型的值。这是完全允许的，因为此时list默认的类型为Object类型。在之后的循环中，由于忘记了之前在list中也加入了String类型的值或其他编码原因，很容易出现类似于//1中的错误。因为编译阶段正常，而运行时会出现“java.lang.ClassCastException”异常。因此，导致此类错误编码过程中不易发现。

那么有没有什么办法可以使集合能够记住集合内元素各类型，且能够达到只要编译时不出现问题，运行时就不会出现“java.lang.ClassCastException”异常呢？答案就是使用泛型。

# 概念

Java在1.5版本中初次加入了泛型。它的本质是参数化类型（Parameterized Type）的应用，也就是说所操作的数据类型被指定为一个参数，在用到的时候在指定具体的类型。

这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口和泛型方法。在平常开发中遇到泛型最多的应该就是在集合部分了。



# 泛型类/接口

## 定义
泛型类(接口)就是具有一个或多个类型变量的类(接口)。对于这个类(接口)来说，只需要关注泛型，而不要为数据存储的细节烦恼。接下来我们使用一个简单的Pair类作为例子，下面是Pair的代码：
```java
public class Pair<T> {
	
	private T first;
	private T second;
	
	public Pair(){}
	
	public Pair(T first,T second){
		this.setFirst(first);
		this.setSecond(second);
	}

	public T getFirst() {
		return first;
	}

	public void setFirst(T first) {
		this.first = first;
	}

	public T getSecond() {
		return second;
	}

	public void setSecond(T second) {
		this.second = second;
	}
}
```

Pair引入了一个类型变量T，用(<>)括起来，放在类名后面。泛型类可以有多个类型变量。例如:
public class Pair<T,U>{…}

泛型中类型变量使用大写形式，并且一般使用单个字符。在Java库中，使用变量E表示集合的元素类型，K和V分别表示关键字和值类型。T表示任意类型。

## 使用

Pair<String> pair=new Pair<String>();

泛型类做父类时：

```java
//① 子类实现时使用泛型变量
public class Person<T> extends Pair<T> {
	...
}
//① 实例化采用下面方式
Person<String> person=new Person<String>();


//② 子类实现时使用具体类型：
public class Person extends Pair<String> {
	...
}
//② 实例化采用下面方式
Person person=new Person();//没有泛型
```

# 泛型方法


```java
public class PrintClass {
	public static <T> void print(T t){
		System.out.println(t.getClass().toString());
	}
}
```
上面print()方法就是一个泛型方法，泛型方法可以定义在普通类中，也可以定义在泛型类中。  
类型变量放在修饰符(public static)后面，返回类型前面。

在调用一个泛型方法时，可以省略类型参数，多数情况下都可以省略，编译器有足够的信息可以判断出所调用的方法类型。

```java
//可以省略类型参数
PrintClass.print(123);
//加入类型参数
PrintClass.<String>print("hello");
```
可以添加多个类型变量，多个类型变量之间用逗号隔开:
```java
public static  <T,S> void print(T t){
	System.out.println(t.getClass().toString());
}
``` 

# 泛型的机制


泛型是编译时的语法，怎么理解这句话呢？泛型只在编译时强化它们的类型信息，并在运行时丢弃它们的类型信息。就是说在编译后的class文件中是没有泛型的任何相关信息的，泛型被擦除了。

在虚拟机中运行的都是普通的类，无论何时定义一个泛型类型，在编译时，都会自动提供一个原始类型。原始类型就是删除类型参数后的泛型类型名。擦除类型变量，并替换为限定类型(无限的变量用Object)。关于限定看后面

上面示例提供的Pair<T>的原始类型:
```java
public class Pair {
	
	private Object first;
	private Object second;
	
	public Pair(){}
	
	public Pair(Object first,Object second){
		this.setFirst(first);
		this.setSecond(second);
	}

	public Object getFirst() {
		return first;
	}

	public void setFirst(Object first) {
		this.first = first;
	}

	public Object getSecond() {
		return second;
	}

	public void setSecond(Object second) {
		this.second = second;
	}
}
```

因为T是一个没有限定的变量，所以用Object替换。原始类型用第一个限定的类型变量来替换，如果没有给定限定就用Object替换。

## 翻译泛型表达式

当程序运行泛型方法时，如果擦除泛型类型，编译器会插入强制类型转换。
```java
Pair<String> pair=new Pair<String>();
String first=pair.getFirst();
```

擦除getFirst返回类型后将会返回Object类型，编译器会自动插入String强制类型转换。也就是说，编译器会把这个方法调用翻译为两条虚拟机指令：

1.对原始方法Pair.getFirst的调用。  
2.将返回Object强制转换为String类型。

## 泛型的约束
1、泛型类型变量必须是类名，不能使基本类型。没有Pair<double>，只有Pair<Double>。

2、运行时类型查询只适用于原始类型。

```java
if(pair instanceof Pair<String>){//错误
	...
}
//或者
if(pair instanceof Pair<T>){//错误
	...
}
```
3、不能参数化类型数组。即Java中没有所谓的泛型数组一说。
```java
///错误
Pair<String> pairs []=new Pair<String>[10];
```
因为泛型会在编译时擦除类型变量，在运行时都是原始类型，而数组会在运行时检查元素类型约束。

4、不能实例化类型变量

不能使用new T()或者T.class这样表达式中的类型变量。

因为类型擦除会将T变成Object，但是开发中本意肯定不是希望new Object()。

5、泛型类的静态上下文中类型变量无效。
```java
public class Singleton<T> {
	private static T instance;//错误
	public static T getInstance(){//错误
		...
	}
}
```

6、不能抛出或者捕获泛型类的实例。


# 泛型类型的继承规则
```java
List<Number> list01=new ArrayList<Number>();		
List<Integer> list02=new ArrayList<Integer>();
list01=list02;//错误

List<Number> list01=new ArrayList<Integer>();//错误
```
Integer是Number的子类，但是list01和list02却没有任何子类和父类之间的关系，简单来说，就是List《T》与List《S》没有任何关系。

如果在使用泛型时，如果两边的泛型变量类型不同也不行，可以这么理解，泛型类型的继承与泛型类之间继承关系是完全不同的。

泛型类可以扩展或者实现其它的泛型类。就这一点而言，与普通类没有什么区别。例如，ArrayList《T》实现List<T>接口。这意味着一个ArrayList《Integer》可以转换为一个List《Integergt》;。但是一个ArrayList《Integer》却不是一个ArrayList《Number》
或List《Number》。


# 泛型通配符
先介绍一个泛型中用到的关键字extends，事实上就是限制了类型变量操作类型的最大父类。现在讲Pair中类型变量上线设置成“Number”类型，那么此时所能接收的类型变量只能是Number及其子类如Integer、Float等。
```java
public class Pair<T extends Number> {
	...
}
Pair<Integer> p1=new Pair<Integer>();
Pair<Float> p2=new Pair<Float>();

Pair<String> ps=new Pair<String>();//错误
```
在泛型类或接口上该种实现方式非常有效，有时候我们自己定义的一个泛型类或者接口，为了缩小传入类型变量的范围，也为了让程序更安全，要求泛型实例中的类型变量一定要是某个类型变量的子类，这时候extends实在是太有用了。

## 上/下限通配符
通配符一般在传参的时候用到，示例如下：
```java
public static void print(Pair<? extends Number> pair){
	System.out.println(pair.getFirst());
}
```

在print方法中传入的是一个泛型类，但是要求泛型类中类型变量必须是Number的子类才可以。这样我们在每次调用该方法的时候，只要传入的类型不是Number子类的泛型类型变量编译器都会报错。程序的可读性大大增强了，代码也更安全了。

extends是类型上限通配符，还有一个类型下限通配符super,在本例中就要这么写：
```java
public static void print(Pair<? super Number> pair){
	System.out.println(pair.getFirst());
}
```
要求传入的类型必须是Number的父类。

## 无限定的通配符

无界通配符不加任何修饰即可，单独一个“？”。如List<?>，“？”可以代表任意类型，“任意”也就是未知类型。
```java
public static void printList(List<?> list) {
    for (Object elem: list)
        System.out.print(elem + "");
    System.out.println();
}
```
```java
List<Integer> li = Arrays.asList(1, 2, 3);
List<String>  ls = Arrays.asList("one", "two", "three");
printList(li);
printList(ls);
```
在Java集合框架中，对于参数值是未知类型的容器类，只能读取其中元素，不能向其中添加元素，因为，其类型是未知，所以编译器无法识别添加元素的类型和容器的类型是否兼容，唯一的例外是null。

无限定的通配符只可以读取，不可使设置任何具体的值，除了可以setXX(null)或者集合元素add(null)之外。

# 进阶
待撸<http://www.jianshu.com/p/4caf2567f91d>

# 参考链接

<http://www.sunnyang.com/423.html>