---
title: Java反射
date: 2016-06-11 22:56:34
categories: Java
tags: 反射
---


# 概念

<!--more-->

是指程序可以访问，检测和修改它本身状态或行为的一种能力，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。（看不懂...）

# 作用

反射可以让我们在运行时获取类的属性，方法，构造方法、父类、接口等信息，通过反射还可以让我们在运行期实例化对象、调用方法、即使方法或属性是私有的的也可以通过反射的形式调用。

# 相关类

java.lang.Class;                
java.lang.reflect.Constructor; 
java.lang.reflect.Field;        
java.lang.reflect.Method;
java.lang.reflect.Modifier;

# Class对象

Java程序在运行时，Java运行时系统一直对所有的对象进行所谓的运行时类型标识。这项信息纪录了每个对象所属的类。虚拟机通常使用运行时类型信息选准正确方法去执行，用来保存这些类型信息的类是Class类。
ClassLoader在需要时根据.class文件内记载的类信息来产生一个与该类相联系的独一无二的Class对象，该Class对象记载了该类的字段，方法等等信息。而java中的Class类对象是可以通过代码主动得到。正是基于此，才有了反射机制。


获取Class对象有三种方式：

- 1.如果T是一个已定义的类型的话，在java中，它的.class文件名：T.class就代表了与其匹配的Class对象，例如：Class c1 = Manager.class(class小写)
- 2.如果有了某个类的对象，可以通过Object类的getClass()方法。例如：
	- Class c2 = new Person().getClass();
- 3.如果你知道类的完整类路径，可以通过Class类的静态方法——forName()来实现,可能会抛出异常。例如：
```java
try {
	Class<?> cls3 = Class.forName("com.sun.study.ui.activity.ReflectionActivity");
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}
```

# 相关方法和示例
方法(通过Class对象调用)：

```java
getName()：获得类的完整名字。  
newInstance()：通过类的不带参数的构造方法创建这个类的一个对象。
注意这个方法只能创建无参数的构造函数的类，如果某类只有带参数的构造函数，那么就要使用另外一种方式：fromClass.getDeclaredConstructor(int.class,String.class).newInstance(24,"wanggc");

//Field
getFields()：获得类的public类型的属性。  
getDeclaredFields()：获得类的所有属性。

//Method
getMethods()：获得类的public类型的方法。  
getDeclaredMethods()：获得类的所有方法。
getDeclaredMethod("方法名",参数类型.class,……)：获得指定的方法  
getMethod(String name, Class[] parameterTypes)：获得类的特定方法。


getModifiers()和Modifier.toString()：获得属修饰符，例如private，public，static等  
getReturnType()：获得方法的返回类型  
getParameterTypes()：获得方法的参数类型

//Constructors
getConstructors()：获得类的public类型的构造方法。 
getConstructor(Class[] parameterTypes)：获得类的特定构造方法。
getDeclaredConstructors():获取所有的构造方法 
getDeclaredConstructor(参数类型.class,……):获得特定的构造方法

getSuperclass()：获取某类的父类的Class对象  
getInterfaces()：获取某类实现的接口的Class对象

getClassLoader()：返回该Class对象对应的类的类加载器。

isArray()
判定此Class对象所对应的是否是一个数组对象。


getComponentType()
该方法针对数组对象的Class对象，可以得到该数组的组成元素所对应对象的Class对象。例如：
int[] ints = new int[]{1,2,3};
Class class1 = ints.getClass();
Class class2 = class1.getComponentType();
而这里得到的class2对象所对应的就应该是int这个基本类型的Class对象。

setAccessible(true):可以破除private的限制

```

栗子：

生成对象

```java
//一、用无参构造：
Class<?> tclass = Reflection2.class;
Object reflection2 = classType.newInstance();

//二、用有参构造：
Class<?> tClass = Person.class;  
Constructor cons = classType.getConstructor(new Class[]{String.class, int.class});   
Object obj = cons.newInstance(new Object[]{“zhangsan”, 19}); 
```


获得类的所有方法（Method）信息：

```java
private void getMethodsInfo() {
    Class<ReflectionActivity> cls = ReflectionActivity.class;
    Method[] methods = cls.getDeclaredMethods();
    if (methods == null) return;

    StringBuilder sb = new StringBuilder();
    for (Method method:methods) {
        sb.append(Modifier.toString(method.getModifiers())).append(" ");
        sb.append(method.getReturnType()).append(" ");
        sb.append(method.getName()).append("(");
        Class[] parameters = method.getParameterTypes();
        if (parameters != null) {
            for (int i=0; i<parameters.length; i++) {
                Class paramCls = parameters[i];
                sb.append(paramCls.getSimpleName());
                if (i < parameters.length - 1) sb.append(", ");
            }
        }
        sb.append(")\n\n");
    }

    tvInfo.setText(sb.toString());
}
```

获取特定属性并设置属性：

```java
//获取类  
Class c = Class.forName("User");  
//获取id属性  
Field idF = c.getDeclaredField("id"); 
//idF.getType:获取属性类型;idF.getName:获取属性名；idF.getModifiers:获取属性修饰符
//实例化这个类赋给o  
Object o = c.newInstance();  
//打破封装  
idF.setAccessible(true); //使用反射机制可以打破封装性，导致了java对象的属性不安全。  
//给o对象的id属性赋值"110"  
idF.set(o, "110"); //set  
//得到设置了属性的实例
System.out.println(idF.get(o));  

```

获取特定方法并使用：

```java
Class c = Cat.class;
Object obj = c.newInstance();//只能创建用无参构造的对象
Method m = c.getDeclaredMethod("eat",String.class);
m.setAccessible(true);//可能是private的
Object result = m.invoke(obj,"fish");//和obj.eat("fish")有点反着来的意思
```


# 实际应用

待补充...


# 参考链接

<http://sunfusheng.com/java/2016/04/12/reflection-annotation-injection.html>
<http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html>

