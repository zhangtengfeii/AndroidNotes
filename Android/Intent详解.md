---
title: Intent详解
date: 2016-06-07 14:56:32
categories: Android
tags: Intent
---

# 定义
Intent是系统各组件之间进行数据传递的数据负载者。当我们需要做一个调用动作，我们就可以通过Intent告诉Android系统来完成这个过程，Intent就是调用通知的一种操作。

<!--more-->

几个常见的操作：  

- 启动一个Activity：Context.startActivity(Intent intent)/startActivityForResult();  
- 启动一个Service：Context.startService(Intent service);
- 绑定一个Service：Context.bindService(Intent service, ServiceConnection conn, int flags);
- 发送一个Broadcast：Context.sendBroadcast(Intent intent);


# 属性
Intent有以下7个组成部分:

- component(组件)：目的组件
- action(动作)：用来表现意图的行为
- category(类别)：用来表现动作的类别
- data(数据):用来表现动作操作的数据
- type(数据类型)：对于data范围的描写
- extras(扩展信息)：扩展信息
- flags(标志位)：Activity的启动方式

## IntentFilter
IntentFilter对象负责过滤掉组件无法响应和处理的Intent，只将自己关心的Intent接收进来进行处理。 IntentFilter实行“白名单”管理，即只列出组件乐意接受的Intent，但IntentFilter只会过滤隐式Intent，显式的Intent会直接传送到目标组件。 Android组件可以有一个或多个IntentFilter，每个IntentFilter之间相互独立，只需要其中一个验证通过则可。除了用于过滤广播的IntentFilter可以在代码中创建外，其他的IntentFilter必须在AndroidManifest.xml文件中进行声明。

IntentFilter中具有和Intent对应的用于过滤Action，Data和Category的字段，一个隐式Intent要想被一个组件处理，必须通过这三个环节的检查。

![](http://www.sunnyang.com/wp-content/uploads/2015/07/intent-filter.png)

如果我们Android系统有多个应用程序的IntentFiletr都可以匹配(忽略优先级策略)，则会弹出一个列表让我们选择。比如如果我们系统安装了多个浏览器的时候。


下面介绍各种属性：


## 1.action
表示要执行的动作，是一个字符串常量。在Intent类中定义了大量的Action常量属性，例如，ACTION_CALL(打电话)、ACTION_SENDtO(发送短信)等。  
我们可以使用setAction()来设置Action属性，使用getAction()来获得Intent的Action属性。    
也可以自己定义action：
```java
<activity android:name=".TargetActivity">  
    <intent-filter>  
        <action android:name="com.scott.intent.action.TARGET"/>  
        <category android:name="android.intent.category.DEFAULT"/>  
    </intent-filter>  
</activity>  

启动：  
Intent intent = new Intent("com.scott.intent.action.TARGET");  
startActivity(intent);  
```

系统也有很多默认的action，通过这些action可以方便的实现一些目的。

```java
public static final String ACTION_MAIN = "android.intent.action.MAIN";    

public static final String ACTION_VIEW = "android.intent.action.VIEW";    

public static final String ACTION_WEB_SEARCH = "android.intent.action.WEB_SEARCH";    

public static final String ACTION_CALL = "android.intent.action.CALL";  
  
``` 

具体应用看后面

## 2.data和extras

即执行动作要操作的数据和传递到目标的附加信息  
一个与浏览器交互的例子
```java
/** 
 * 打开指定网页 
 * @param view 
 */  
public void invokeWebBrowser(View view) {  
    Intent intent = new Intent(Intent.ACTION_VIEW);  
    intent.setData(Uri.parse("http://www.google.com.hk"));  
    startActivity(intent);  
}  
  
/** 
 * 进行关键字搜索 
 * @param view 
 */  
public void invokeWebSearch(View view) {  
    Intent intent = new Intent(Intent.ACTION_WEB_SEARCH);  
    intent.putExtra(SearchManager.QUERY, "android");    //关键字  
    startActivity(intent);  
}  
```
上面两个方法分别是启动浏览器并打开指定网页、进行关键字搜索，分别对应的action是Intent.ACTION_VIEW和Intent.ACTION_WEB_SEARCH，前者需指定相应的网页地址，后者需指定关键字信息，对于关键字搜索来说，浏览器会按照自己设置的默认的搜索引擎进行搜索。

我们注意到，在打开网页时，为Intent指定一个data属性，这其实是指定要操作的数据，是一个URI的形式，我们可以将一个指定前缀的字符串转换成特定的URI类型，如：“http:”或“https:”表示网络地址类型，“tel:”表示电话号码类型，“mailto:”表示邮件地址类型，等等。例如，我们要呼叫给定的号码，可以这样做：
```java
public void call(View view) {  
    Intent intent = new Intent(Intent.ACTION_CALL);  
    intent.setData(Uri.parse("tel:12345678"));  
    startActivity(intent);  
}  
```
那么我们如何知道目标是否接受这种前缀呢？这就需要看一下目标中<data/>元素的匹配规则了。
在目标<data/>标签中包含了以下几种子元素，他们定义了url的匹配规则：
android:scheme 匹配url中的前缀，除了“http”、“https”、“tel”...之外，我们可以定义自己的前缀
android:host 匹配url中的主机名部分，如“google.com”，如果定义为“*”则表示任意主机名
android:port 匹配url中的端口
android:path 匹配url中的路径
我们改动一下TargetActivity的声明信息：
```java
<activity android:name=".TargetActivity">  
    <intent-filter>  
        <action android:name="com.scott.intent.action.TARGET"/>  
        <category android:name="android.intent.category.DEFAULT"/>  
        <data android:scheme="scott" android:host="com.scott.intent.data" android:port="7788" android:path="/target"/>  
    </intent-filter>  
</activity>  
```
这个时候如果只指定action就不够了，我们需要为其设置data值，如下:
```java
public void gotoTargetActivity(View view) {  
    Intent intent = new Intent("com.scott.intent.action.TARGET");  
    intent.setData(Uri.parse("scott://com.scott.intent.data:7788/target"));  
    startActivity(intent);  
}  
```
url中的每个部分和TargetActivity配置信息中全部一致才能跳转成功，否则就被系统拒绝。

不过有时候对path限定死了也不太好，比如我们有这样的url：（scott://com.scott.intent.data:7788/target/hello）（scott://com.scott.intent.data:7788/target/hi）
这个时候该怎么办呢？我们需要使用另外一个元素：android:pathPrefix，表示路径前缀。
我们把android:path="/target"修改为android:pathPrefix="/target"，然后就可以满足以上的要求了。

在进行搜索时，我们使用了一个putExtra方法，将关键字做为参数放置在Intent中，我们成为extras（附加信息），这里面涉及到了一个Bundle对象。
Bundle和Intent有着密不可分的关系，主要负责为Intent保存附加参数信息，它实现了android.os.Paracelable接口，内部维护一个Map类型的属性，用于以键值对的形式存放附加参数信息。在我们使用Intent的putExtra方法放置附加信息时，该方法会检查默认的Bundle实例为不为空，如果为空，则新创建一个Bundle实例，然后将具体的参数信息放置到Bundle实例中。我们也可以自己创建Bundle对象，然后为Intent指定这个Bundle即可，如下：
```java
public void gotoTargetActivity(View view) {  
    Intent intent = new Intent("com.scott.intent.action.TARGET");  
    Bundle bundle = new Bundle();  
    bundle.putInt("id", 0);  
    bundle.putString("name", "scott");  
    intent.putExtras(bundle);  
    startActivity(intent);  
}  
```

需要注意的是，在使用putExtras方法设置Bundle对象之后，系统进行的不是引用操作，而是复制操作，所以如果设置完之后再更改bundle实例中的数据，将不会影响Intent内部的附加信息。那我们如何获取设置在Intent中的附加信息呢？与之对应的是，我们要从Intent中获取到Bundle实例，然后再从中取出对应的键值信息：
```java
Bundle bundle = intent.getExtras();  
int id = bundle.getInt("id");  
String name = bundle.getString("name");
```

当然我们也可以使用Intent的getIntExtra和getStringExtra方法获取，其数据源都是Intent中的Bundle类型的实例对象。
前面我们涉及到了Intent的三个属性：action、data和extras。除此之外，Intent还包括以下属性：

## 3.category
要执行动作的目标所具有的特质或行为归类，Category可以添加多个，所以方法是addCategory()。  

例如：在我们的应用主界面Activity通常有如下配置：
```java
<category android:name="android.intent.category.LAUNCHER" /> 
```

代表该目标Activity是该应用所在task中的初始Activity并且出现在系统launcher的应用列表中。


几个常见的category如下：
Intent.CATEGORY_DEFAULT（android.intent.category.DEFAULT） 默认的category
Intent.CATEGORY_PREFERENCE（android.intent.category.PREFERENCE） 表示该目标Activity是一个首选项界面；
Intent.CATEGORY_BROWSABLE（android.intent.category.BROWSABLE）指定了此category后，在网页上点击图片或链接时，系统会考虑将此目标Activity列入可选列表，供用户选择以打开图片或链接。

系统在调用startActivity或者startAcitvityForResult的时候会默认加上"android.intent.category.DEFAULT"，所以为了Activity能够接收隐式调用，配置多个category的时候必须加上默认的category。



##　4.type
要执行动作的目标Activity所能处理的MIME数据类型，type的配置在清单文件AndroidManifest.xml是配置在data中的，作为data属性之一
例如：一个可以处理图片的目标Activity在其声明中包含这样的mimeType：
```java
<data android:mimeType="image/*" />  
```
在使用Intent进行匹配时，我们可以使用setType(String type)或者setDataAndType(Uri data, String type)来设置mimeType。

要注意的是，如果设置了Data就会忽略Type，同样如果设置了Type也会忽略Data，如果你要同时设置data和type，只能用setDataAndType(Uri data， String type)。

```java
public Intent setData(Uri data) {
	mData = data;
	mType = null;
	return this;
}
 
public Intent setType(String type) {
	mData = null;
	mType = type;
	return this;
}
```

一旦我们在清单文件中指定了data，type，在Intent启动组件的时候就必须设置data或type，否则一定不会通过。具体规则如下：

- 如果Intent没有指定data和data type，则只有没有定义data和datetype的filter才能通过测试;
- 如果intent定义了data没有定义datatype，则只有定义了相同data且没有定义datetype的filter才能通过测试;
- 如果intent没有定义data却定义了datatype，则只有未定义data且定义了相同的datatype的filter才能通过测试;
- 如果intent既定义了data也定义了datatype， 则只有定义了相同的data和datatype的filter才能通过测试。  
（好烦啊。。）
## 5.component
显示启动Activity的时候用到的该属性，一旦指定了该属性其它属性都是可选的，如果同时指定了显示启动和隐式启动，那么优先采用显示启动。

在使用component进行匹配时，一般采用以下几种形式：
```java
intent.setComponent(new ComponentName(getApplicationContext(), TargetActivity.class));  //类名.class
intent.setComponent(new ComponentName(getApplicationContext(), "com.scott.intent.TargetActivity"));  //完整包名类名
intent.setComponent(new ComponentName("com.scott.other", "com.scott.other.TargetActivity"));  //其他应用的时候
```

其中，前两种是用于匹配同一包内的目标，第三种是用于匹配其他包内的目标。需要注意的是，如果我们在Intent中指定了component属性，系统将不会再对action、data/type、category进行匹配。

## 6.Flags

启动Activity是它决定了Activity的运行模式，在AndroidManifest.xml中的标签的android:launchMode属性中设置。有四种启动模式，这里就不说了。

# Intent数据传递
两种形式，一种直接传递通过intent.putExtra，另一种是通过Bundle对象来传递，更深入一点事实上都是通过Bundle传递的，也是说当我们采用第一种的时候底层还是采用的Bundle，再深入就是他们都是通过Map对象传递数据的。
```java
	Intent intent = new Intent("com.scott.intent.action.TARGET");  
    Bundle bundle = new Bundle();  
    bundle.putInt("id", 0);  
    bundle.putString("name", "scott");  
    intent.putExtras(bundle);  
    startActivity(intent);  
```

按传递数据的类型分：

- 简单或基本数据类型
- 传递复杂数据类型
- 传递Serializable对象
- Parcelable对象



前面1和2数据传递方式都很简单，Serializable方式和Parcelable方式的不同重点在于两者的存储媒介不同，Serializable使用IO读写存储在硬盘上，而Parcelable是直接在内存中读写，很明显内存的读写速度通常大于IO读写，所以在Android中通常优先选择Parcelable。  

Serializable和Parcelable选择的原则：

- 在使用内存的时候，Parcelable比Serializable性能高，所以推荐使用Parcelable。
- Parcelable不能使用在要将数据存储在磁盘上的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点，但此时还是建议使用Serializable 。

##　Serializable
Serializable方式只需在创建javabean时直接实现Serializable接口就可以了。接着传递和拿取：

```java
传：intent.putExtra("key",person);
拿：Personperson=(Person)getIntent().getSerializableExtra("key");
``` 
## Parcelable
步骤：

1. implements Parcelable
2. 重写writeToParcel方法，将你的对象序列化为一个Parcel对象，即：将类的数据写入外部提供的Parcel中，打包需要传递的数据到Parcel容器保存，以便从 Parcel容器获取数据
3. 重写describeContents方法，内容接口描述，默认返回0就可以
4. 实例化静态内部对象CREATOR实现接口Parcelable.Creator

代码：
```java
public class Person implements Parcelable{

private String name;
private int age;

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}

public int getAge() {
    return age;
}

public void setAge(int age) {
    this.age = age;
}
@Override
public int describeContents() {
    //返回0即可
    return 0;
}

@Override
public void writeToParcel(Parcel dest, int flags) {
    //将自定义的类中的字段用write方法一一写出，什么类型就用对应的write方法
    dest.writeString(name);
    dest.writeInt(age);
}

//提供一个名为CREATOR的常量，并制定泛型为Person
public static final Parcelable.Creator<Person> CREATOR = new Parcelable.Creator<Person>() {
    @Override
    public Person createFromParcel(Parcel source) {
        //读取刚才write出的字段，注意顺序要完全一致！
        Person person = new Person();
        person.name = source.readString();
        person.age = source.readInt();
        return person;
    }
    @Override
    public Person[] newArray(int size) {
        return new Person[size];
    }
};
}
```			
传：intent.putExtra("key",person);  
拿：Personperson=(Person)getIntent().getParcelableExtra("key");

# 代码片段

## 打电话
```java
Uri uri = Uri.parse("tel:10086");
Intent intent = new Intent(Intent.ACTION_DIAL, uri);
startActivity(intent);
```
## 发短信
```java
// 给10086发送内容为“Hello”的短信
Uri uri = Uri.parse("smsto:10086");
Intent intent = new Intent(Intent.ACTION_SENDTO, uri);
intent.putExtra("sms_body", "Hello");
startActivity(intent);
// 发送彩信（相当于发送带附件的短信）
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra("sms_body", "Hello");
Uri uri = Uri.parse("content://media/external/images/media/23");
intent.putExtra(Intent.EXTRA_STREAM, uri);
intent.setType("image/png");
startActivity(intent);
```


## 打开网页
```java
// 打开Google主页
Uri uri = Uri.parse("http://www.google.com");
Intent intent  = new Intent(Intent.ACTION_VIEW, uri);
startActivity(intent);
```

## 发邮件
```java
// 给someone@domain.com发邮件
Uri uri = Uri.parse("mailto:someone@domain.com");
Intent intent = new Intent(Intent.ACTION_SENDTO, uri);
startActivity(intent);
// 给someone@domain.com发邮件发送内容为“Hello”的邮件
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, "someone@domain.com");
intent.putExtra(Intent.EXTRA_SUBJECT, "Subject");
intent.putExtra(Intent.EXTRA_TEXT, "Hello");
intent.setType("text/plain");
startActivity(intent);
// 给多人发邮件
Intent intent=new Intent(Intent.ACTION_SEND);
String[] tos = {"1@abc.com", "2@abc.com"}; // 收件人
String[] ccs = {"3@abc.com", "4@abc.com"}; // 抄送
String[] bccs = {"5@abc.com", "6@abc.com"}; // 密送
intent.putExtra(Intent.EXTRA_EMAIL, tos);
intent.putExtra(Intent.EXTRA_CC, ccs);
intent.putExtra(Intent.EXTRA_BCC, bccs);
intent.putExtra(Intent.EXTRA_SUBJECT, "Subject");
intent.putExtra(Intent.EXTRA_TEXT, "Hello");
intent.setType("message/rfc822");
startActivity(intent);
```

## 播放多媒体
```java
Intent intent = new Intent(Intent.ACTION_VIEW);
Uri uri = Uri.parse("file:///sdcard/foo.mp3");
intent.setDataAndType(uri, "audio/mp3");
startActivity(intent);
 
Uri uri = Uri.withAppendedPath(MediaStore.Audio.Media.INTERNAL_CONTENT_URI, "1");
Intent intent = new Intent(Intent.ACTION_VIEW, uri);
startActivity(intent);
```

## 拍照
```java
//打开拍照程序
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE); 
startActivityForResult(intent, 0);
// 取出照片数据
Bundle extras = intent.getExtras(); 
Bitmap bitmap = (Bitmap) extras.get("data");
```

## 安装卸载程序
```java
Uri uri = Uri.fromParts("package", "com.demo.app", null);  
Intent intent = new Intent(Intent.ACTION_DELETE, uri);  
startActivity(intent);
```

## 进入设置页面
```java
// 进入无线网络设置界面（其它可以举一反三）  
Intent intent = new Intent(android.provider.Settings.ACTION_WIRELESS_SETTINGS);  
startActivityForResult(intent, 0);
```

## 获取裁剪图片
```java
// 获取并剪切图片
Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
intent.setType("image/*");
intent.putExtra("crop", "true"); // 开启剪切
intent.putExtra("aspectX", 1); // 剪切的宽高比为1：2
intent.putExtra("aspectY", 2);
intent.putExtra("outputX", 20); // 保存图片的宽和高
intent.putExtra("outputY", 40); 
intent.putExtra("output", Uri.fromFile(new File("/mnt/sdcard/temp"))); // 保存路径
intent.putExtra("outputFormat", "JPEG");// 返回格式
startActivityForResult(intent, 0);
// 剪切特定图片
Intent intent = new Intent("com.android.camera.action.CROP"); 
intent.setClassName("com.android.camera", "com.android.camera.CropImage"); 
intent.setData(Uri.fromFile(new File("/mnt/sdcard/temp"))); 
intent.putExtra("outputX", 1); // 剪切的宽高比为1：2
intent.putExtra("outputY", 2);
intent.putExtra("aspectX", 20); // 保存图片的宽和高
intent.putExtra("aspectY", 40);
intent.putExtra("scale", true);
intent.putExtra("noFaceDetection", true); 
intent.putExtra("output", Uri.parse("file:///mnt/sdcard/temp")); 
startActivityForResult(intent, 0);
```

# 参考链接
<http://blog.csdn.net/liuhe688/article/details/7162988>
http://www.sunnyang.com/239.html