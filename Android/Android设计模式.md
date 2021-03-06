---
title: Android单例模式
date: 2016-06-11 21:42:29
categories: Android
tags: 设计模式
---

# 单例模式

<!--more-->


单例模式有以下特点：

- 1、单例类只能有一个实例。
- 2、单例类必须自己创建自己的唯一实例。
- 3、单例类必须给所有其他对象提供这一实例。

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。

## 一、懒汉式

```java
//懒汉式单例类.在第一次调用的时候实例化自己   
public class Singleton {  
	private static Singleton single; 
    private Singleton() {}  
    
    public static Singleton getInstance() {  
         if (single == null) {    
             single = new Singleton();  
         }    
        return single;  
    }  
}  
```

缺点：线程不安全，并发环境下很可能出现多个Singleton实例。
优点：相对于饿汉式来说，需要时才加载可以减少资源消耗。


## 二、饿汉式

```java
//饿汉式单例类.在类初始化时，已经自行实例化   
public class Singleton {  
	private static Singleton instance = new Singleton();
    private Singleton() {}       
    public static Singleton getInstance() {  
        return instance;  
    }  
}  
```

饿汉式在类创建的同时就已经创建好一个静态的对象供系统使用，以后不再改变，所以天生是线程安全的。  

这种方式简单粗暴，如果单例对象初始化非常快，而且占用内存非常小的时候这种方式是比较合适的，可以直接在应用启动时加载并初始化。  
但是，如果单例初始化的操作耗时比较长而应用对于启动速度又有要求，或者单例的占用内存比较大，再或者单例只是在某个特定场景的情况下才会被使用，而一般情况下是不会使用时，使用「饿汉式」的单例模式就是不合适的，这时候就需要用到「懒汉式」的方式去按需延迟加载单例。

## 三、懒汉式-加同步锁

```java
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static synchronized Singleton getInstance() {  
	    if (instance == null) {  
	        instance = new Singleton();  
	    }  
    return instance;  
    }  
}  
```
线程安全了，但是效率很低。。


## 四、懒汉式-双重检查锁定

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
	    if (singleton == null) {  
	        synchronized (Singleton.class) {  
		        if (singleton == null) {  
		            singleton = new Singleton();  
		        }  
	        }  
	    }  
    return singleton;  
    }  
}
```
在 synchronized (Singleton.class) 外又添加了一层if，这是为了在instance已经实例化后下次进入不必执行 synchronized (Singleton.class) 获取对象锁，从而提高性能。

## 五、静态内部类

```java
public class Singleton {  
	//内部类，在加载内部类时才回去创建单例
    private static class SingletonHolder {  
    	private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    	return SingletonHolder.INSTANCE;  
    }  
}  
```

Singleton类被装载了，instance不一定被初始化。因为SingletonHolder类没有被主动使用，只有显示通过调用getInstance方法时，才会显示装载SingletonHolder类，从而实例化instance。

## 六、枚举

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {
		//do something...  
    }  
}  
```

使用：

```java
public static void main(String[] args){
	Singleton instance = Singleton.INSTANCE;
	instance.whateverMethod();	
}


```

这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。不过因为枚举是1.5才加入的特性，


## 选择
一般来说，不会选择第一种。剩下5种：饿汉，懒汉(加同步锁)，双重校验锁，枚举和静态内部类。
如果没什么特殊要求，用饿汉。
明确需要lazy loading时，用静态内部类。
涉及到反序列化创建对象时，用枚举方式。



# 待撸

<http://www.jianshu.com/p/bdf65e4afbb0>  
<https://github.com/simple-android-framework-exchange/android_design_patterns_analysis>

# 参考链接

<http://cantellow.iteye.com/blog/838473>