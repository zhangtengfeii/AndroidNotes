---
title: Fragment知识点
date: 2016-06-10 12:54:02
categories: Android 
tags: Fragment
---



同样的界面Activity占用内存比Fragment要多，响应速度Fragment比Activty在中低端手机上快了很多，甚至能达到好几倍！如果你的app当前或以后有移植平板等平台时，可以让你节省大量时间和精力。

<!--more-->


# 创建

Fragment是在Android3.0才引入的，所以如果我们的应用还要兼容3.0之前的版本的话，我们可以使用support v4包。有两种创建方式，静态添加和动态添加。




## 静态添加

这是使用Fragment最简单的一种方式，把Fragment当成普通的控件，直接写在布局文件中。步骤：

1. 新建一个类Fragment1继承Fragment，重写onCreateView决定Fragemnt的布局,当然了该Fragment1的布局（R.layout.fragment1）要你自己创建好。

```java
public class Fragment1 extends Fragment {  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {  
        return inflater.inflate(R.layout.fragment1, container, false);  
    }  
  
}  
```
2. 在Activity的布局文件中使用<fragment/>标签，通过name属性指定对应的Fragment类（要写全类名）。
```java
<fragment  
	android:id="@+id/fragment1"  
	android:name="com.example.fragmentdemo.Fragment1"  
	android:layout_width="0dip"  
	android:layout_height="match_parent"  
	android:layout_weight="1" />  
```
3. 在Activity中findViewById找到Fragment，就当和普通的View一样，就可以进行各种操作了



## 动态添加
步骤：

1. 获取到FragmentManager，在Activity中可以直接通过getFragmentManager得到。
2. 开启一个事务，通过调用beginTransaction方法开启。
3. 向容器（一般为FrameLayout）内加入Fragment，一般使用replace或者add方法实现，需要传入容器的id和Fragment的实例。
4. 提交事务，调用commit方法提交。


```java
FragmentManager mgr = getSupportFragmentManager();
FragmentTransaction tx = mgr.beginTransaction();
Fragment fragment=new Fragment01();
tx.add(R.id.layout_right, fragment);//参数为容器的id
//tx.addToBackStack(null);
tx.commit();
```

# 常用的类

- Fragment 主要用于定义Fragment
- FragmentManager 主要用于在Activity中操作Fragment
- FragmentTransaction 保证一些列Fragment操作的原子性，熟悉事务这个词，一定能明白~

FragmentManager通过getFragmentManager() 或者getSupportFragmentManager获得，它的常用方法：

- beginTransaction()：开启一个事务；
- addOnBackStackChangedListener(OnBackStackChangedListener listener)：返回键时回调函数；
- findFragmentById (int id)：根据id查找返回一个Fragment, 适用于在布局中提供了UI的Fragment；
- findFragmentByTag (String tag)：根据标签名查找返回一个Fragment，对于提供或没有提供UI的Fragment都适用；
- popBackStack()：将片段从返回栈中弹出（模拟一次用户按下BACK键的指令）


FragmentTransaction提供了各种对Fragment的操作，每执行一次事务都代表了对Fragment执行了一次操作。比较常见的更新Fragment的操作有show、hide、add、remove、replace。

1.获取：
FragmentTransaction transaction = fm.benginTransatcion();//开启一个事务

2.添加Fragment：
transaction.add(fragment) 

3.移除：
transaction.remove(fragment) 

4.替换：
transaction.replace(R.id.framelayout,fragment)  
replace的话会调用Fragment的生命周期，也就是说它会销毁视图，重新加载，这种方式的话如果你的Fragment里面有大量的数据或者说很多视图结构的话不推荐使用这种，会增大你的内存消耗。这种情况建议用hide，show 

5.隐藏（不可见，并不会销毁）：
transaction.hide(fragment)  
会回调onHiddenChanged这个方法，可以在里面做一些轻量级操作。

6.显示隐藏的：
transaction.show(fragment)

7.提交：
transatcion.commit()

对于每一个Fragment在执行commit之前都可以执行一次动画，记住一点就是commit执行后并没有立即执行事务操作，它只是通知UI主线程可以调度这次事务。如有必要，可以在UI线程中调用executePendingTransactions()来立即执行由commit()提交的事务。

8.detach()
会将view从UI中移除,和remove()不同,此时fragment的状态依然由FragmentManager维护。

9.attach()
重建view视图，附加到UI上并显示。

# 管理Fragment回退栈

类似与Android系统为Activity维护一个任务栈，我们也可以通过Activity维护一个回退栈来保存每次Fragment事务发生的变化。如果你将Fragment任务添加到回退栈，当用户点击后退按钮时，将看到上一次的保存的Fragment。一旦Fragment完全从后退栈中弹出，用户再次点击后退键，则退出当前Activity。

如何添加一个Fragment事务到回退栈：  
FragmentTransaction.addToBackStack(String)

其中String是可选值,一般写null，这个操作是将一次事务操作放入回退栈（保留本次commit前的那个Fragment实例），如果在平板电脑中操作复杂界面，就如同在浏览器中点击回退按钮一样，只要页面没有被系统回收，总会返回当前操作的上一次操作。该方法仅仅是将一次事务操作放入回退栈中，而不是将某个特定Fragment放入回退栈或者销毁。看下面的例子：

```java
tx.replace(R.id.left, fragment01, "fragment01");
tx.addToBackStack("fragment01");
tx.replace(R.id.right, fragment02, "fragment02");
tx.addToBackStack("fragment02");
```
上面的代码切换了两个Fragment,我们按一次back键，fragment01和fragment02都会返回消失，因为这两个Fragment是在同一个事务中进行操作的。

用replace()后，如果该事务加入到了回退栈，Fragment实例就不会被销毁，只是视图销毁了；
这里的视图销不销毁，指的是我们的数据有没有被保存下来。  

当然，在Fragment里面也有onSaveInstanceState(Bundle)方法，可以通过这个来保存数据，然后再onCreate或其他方法里面获取到数据来解决数据丢失的问题。




# 生命周期
Fragment是嵌套在Activity中的，因此它的生命周期与Activity息息相关  
![](http://www.sunnyang.com/wp-content/uploads/2015/08/activity_fragment_lifecycle.png)

可以看到Fragment比Activity多了几个额外的生命周期回调方法：  

- onAttach(Activity)
当Fragment与Activity发生关联时调用。
- onCreateView(LayoutInflater, ViewGroup,Bundle)
创建该Fragment的视图
- onActivityCreated(Bundle)
当Activity的onCreate方法返回时调用
- onDestoryView()
与onCreateView想对应，当该Fragment的视图被移除时调用
- onDetach()
与onAttach相对应，当Fragment与Activity关联被取消时调用

注意：除了onCreateView，其他的所有方法如果你重写了，必须调用父类对于该方法的实现(就是super())


启动Activity时：

![](http://www.sunnyang.com/wp-content/uploads/2015/08/fragment_start_activity.png)


销毁Activity时：

![](http://www.sunnyang.com/wp-content/uploads/2015/08/fragment_destory_activity.png)

针对Activity状态的改变Fragment状态的改变就如果入栈出栈的操作，Activity启动的时候相应的Fragment状态总是后执行，当我们要销毁Activity时，Fragment的状态总是优先销毁。就如同进栈的时候Activity先进入，出栈的时候Activity后出，先进后出，恰好符合栈的操作。


# Fragment与Activity通信

最基本的方法：

- a、如果你Activity中包含自己管理的Fragment的引用，可以通过引用直接访问所有的Fragment的public方法
- b、如果Activity中未保存任何Fragment的引用，那么没关系，每个Fragment都有一个唯一的TAG或者ID,可以通过getFragmentManager.findFragmentByTag()或者findFragmentById()获得任何Fragment实例，然后进行操作。
- c、在Fragment中可以通过getActivity得到当前绑定的Activity的实例，然后进行操作。  
注：如果在Fragment中需要Context，可以通过调用getActivity(),如果该Context需要在Activity被销毁后还存在，则使用getActivity().getApplicationContext()。

但是实际使用中，还要考虑Fragment的重复使用，所以必须降低Fragment与Activity的耦合，而且Fragment更不应该直接操作别的Fragment，Fragment操作应该由它的管理者Activity来决定。

### Activity传值到Fragment

Fragment中这么写：

```java
public class ContentFragment extends Fragment  
{  
  
    private String mArgument;  
    public static final String ARGUMENT = "argument";  
  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        // mArgument = getActivity().getIntent().getStringExtra(ARGUMENT);  
        Bundle bundle = getArguments();  
        if (bundle != null)  
            mArgument = bundle.getString(ARGUMENT);  
  
    }  
  
    /** 
     * 传入需要的参数，设置给arguments 
     * @param argument 
     * @return 
     */  
    public static ContentFragment newInstance(String argument)  
    {  
        Bundle bundle = new Bundle();  
        bundle.putString(ARGUMENT, argument);  
        ContentFragment contentFragment = new ContentFragment();  
        contentFragment.setArguments(bundle);  
        return contentFragment;  
    }  
```
Activity中这么写：

```java
ContentFragment contentFragment= ContentFragment.newInstance("str");
```

### Fragment主动传值到Activity
随意一点的话直接在Fragment中getActivity。

最常见的是采用接口方案

```java
//MainActivity实现MainFragment开放的接口 
public class MainActivity extends FragmentActivity implements FragmentListener{ 
    @override
     public void toH5Page(){ }//实现接口中的方法
   
} 

public class MainFragment extends Fragment{
	public FragmentListener mListener;  
    
	@Override 
    public void onAttach(Activity activity) { 
          super.onAttach(activity); 
          //对传递进来的Activity进行接口转换
           if(activity instanceof FragmentListener){
               mListener = ((FragmentListener)activity); 
          }
     }
	//比如点击一个按钮后就跳转到H5页面：mListener.toH5Page()

	//定义一个接口，接口中定义一些用户操作Fragment后，希望Activity作出响应的方法，当然也可以通过这个回调传递参数
    public static interface FragmentListener{ 
        //跳到h5页面
       void toH5Page();
     }

}
```
这种方案应该是既能达到复用，又能达到很好的可维护性，并且性能也是杠杠的。但是唯一的一个遗憾是假如项目很大了，Activity与Fragment的数量也会增加，这时候为每对Activity与Fragment交互定义交互接口就是一个很头疼的问题（包括为接口的命名，新定义的接口相应的Activity还得实现，相应的Fragment还得进行强制转换）。




...
# DialogFragment
...
<http://blog.csdn.net/lmj623565791/article/details/37815413>


# Fragment实例销毁

一定要搞清楚哪个方法会销毁视图，哪个会销毁实例，哪个仅仅只是隐藏，这样才能更好的使用它们。

- a、比如：我在FragmentA中的EditText填了一些数据，当切换到FragmentB时，如果希望会到A还能看到数据，则适合你的就是hide(this)和add(newFragment)；也就是说，希望保留用户操作的面板，你可以使用hide和show，当然了不要使劲在那new实例，进行下非null判断。
- b、再比如：我不希望保留用户操作，你可以使用remove()，然后add()；或者使用replace()这个和remove,add是相同的效果。
- c、remove和detach有一点细微的区别，在不考虑回退栈的情况下，remove会销毁整个Fragment实例，而detach则只是销毁其视图结构，实例并不会被销毁。那么二者怎么取舍使用呢？如果你的当前Activity一直存在，那么在不希望保留用户操作的时候，你可以优先使用detach。



# 切换动画：
```java
FragmentTransaction fragmentTransaction = manager.beginTransaction();
fragmentTransaction
        .setCustomAnimations(R.animator.fragment_slide_left_enter, R.animator.fragment_slide_left_exit,
                R.animator.fragment_slide_right_enter, R.animator.fragment_slide_right_exit)
        .replace(ResID, fragment, s)
		.addToBackStack(s)
        .commit();
```
给Fragment设定Fragment转场动画时，如果你的app有使用popStackBackxx(tag/id,flags)出栈多个Fragment时，应避免直接使用.setCustomAnimations(enter, exit, popEnter, popExit)，需要配合onCreateAnimtation方法将出栈动画临时取消


# getActivity()空指针

大多数情况下的原因：你在调用了getActivity()时，当前的Fragment已经onDetach()了宿主Activity。  
比如：你在pop了Fragment之后，该Fragment的异步任务仍然在执行，并且在执行完成后调用了getActivity()方法，这样就会空指针。

更安全的做法：在Fragment基类里设置一个Activity mActivity的全局变量，在onAttach(Activity activity)里赋值，使用mActivity代替getActivity()，在onDetach中将mActivity置为null（不然会引起内存泄露）


# Fragment重叠异常

安卓app有一种特殊情况，就是 app运行在后台的时候，系统资源紧张的时候导致把app的资源全部回收（杀死app的进程），这时把app再从后台返回到前台时，app会重启。这种情况下文简称为：“内存重启”。（屏幕旋转等配置变化也会造成当前Activity重启，本质与“内存重启”类似）

在系统要把app回收之前，系统会把Activity的状态保存下来，Activity的FragmentManager负责把Activity中的Fragment保存起来。在“内存重启”后，Activity的恢复是从栈顶逐步恢复，Fragment会在宿主Activity的onCreate方法调用后紧接着恢复（从onAttach生命周期开始）。

如果你add()了几个Fragment，使用show()、hide()方法控制，比如微信、QQ的底部tab等情景，如果你什么都不做的话，在“内存重启”后回到前台，app的这几个Fragment界面会重叠。

原因是FragmentManager帮我们管理Fragment，当发生“内存重启”，他会从栈底向栈顶的顺序一次性恢复Fragment；

但是因为没有保存Fragment的mHidden属性，默认为false，即show状态，所以所有Fragment都是以show的形式恢复，我们看到了界面重叠。
（如果是replace，恢复形式和Activity一致，只有当你pop之后上一个Fragment才开始重新恢复，所有使用replace不会造成重叠现象）

解决方法有两种：findFragmentByTag、getFragments()

### findFragmentByTag

即在add()或者replace()时绑定一个tag，一般我们是用fragment的类名作为tag，然后在发生“内存重启”时，通过findFragmentByTag找到对应的Fragment，并hide()需要隐藏的fragment。

下面是个标准恢复写法：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity);

    TargetFragment targetFragment;
    HideFragment hideFragment;

    if (savedInstanceState != null) {  // “内存重启”时调用
        targetFragment = getSupportFragmentManager().findFragmentByTag(TargetFragment.class.getName);
        hideFragment = getSupportFragmentManager().findFragmentByTag(HideFragment.class.getName);
        // 解决重叠问题
        getFragmentManager().beginTransaction()
                .show(targetFragment)
                .hide(hideFragment)
                .commit();
    }else{  // 正常时
        targetFragment = TargetFragment.newInstance();
        hideFragment = HideFragment.newInstance();

        getFragmentManager().beginTransaction()
                .add(R.id.container, targetFragment, targetFragment.getClass().getName())
                .add(R.id,container,hideFragment,hideFragment.getClass().getName())
                .hide(hideFragment)
                .commit();
    }
}
如果你想恢复到用户离开时的那个Fragment的界面，你还需要在onSaveInstanceState(Bundle outState)里保存离开时的那个可见的tag或下标，在onCreate“内存重启”代码块中，取出tag/下标，进行恢复。
```

### 通过getFragments

通过getFragments()可以获取到当前FragmentManager管理的栈内所有Fragment。

标准写法如下：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity);

    TargetFragment targetFragment;
    HideFragment hideFragment;

    if (savedInstanceState != null) {  // “内存重启”时调用
        List<Fragment> fragmentList = getSupportFragmentManager().getFragments();
        for (Fragment fragment : fragmentList) {
            if(fragment instanceof TartgetFragment){
               targetFragment = (TargetFragment)fragment; 
            }else if(fragment instanceof HideFragment){
               hideFragment = (HideFragment)fragment;
            }
        ｝
        // 解决重叠问题
        getFragmentManager().beginTransaction()
                .show(targetFragment)
                .hide(hideFragment)
                .commit();
    }else{  // 正常时
        targetFragment = TargetFragment.newInstance();
        hideFragment = HideFragment.newInstance();

        // 这里add时，tag可传可不传
        getFragmentManager().beginTransaction()
                .add(R.id.container)
                .add(R.id,container,hideFragment)
                .hide(hideFragment)
                .commit();
    }
}
```

### putFragment
```java
// 保存
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);

    getSupportFragmentManager().putFragment(outState, KEY, targetFragment);
}
// 恢复
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_scrolling);

    if (savedInstanceState != null) {
        Fragment targetFragment = getSupportFragmentManager().getFragment(savedInstanceState, KEY);
    }
｝
```
如果仅仅为了找回栈内的Fragment，使用putFragment(bundle, key, fragment)保存fragment，是完全没有必要的；因为FragmentManager在任何情况都会帮你存储Fragment，你要做的仅仅是在“内存重启”后，找回这些Fragment即可。

# 运行时配置发生变化
这里还有一种情况会发生重叠。  
在Activity的onCreate中new Fragment然后add到FrameLayout，如果手机配置发生改变（如屏幕旋转），Activity会发生重新启动，默认的Activity中的Fragment也会跟着Activity重新创建，也就是说：本身存在的Fragment会重新启动，然后当执行Activity的onCreate时，又会再次实例化一个新的Fragment，这也会发生重叠现象。

如何解决?先判断一下savedInstanceState是否为null再决定是否new Fragment。
```java
public class MainActivity extends Activity  
  
{  
    private static final String TAG = "FragmentOne";  
    private FragmentOne mFOne;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        setContentView(R.layout.activity_main);  
  
        Log.e(TAG, savedInstanceState+"");  
          
        if(savedInstanceState == null)  
        {  
            mFOne = new FragmentOne();  
            FragmentManager fm = getFragmentManager();  
            FragmentTransaction tx = fm.beginTransaction();  
            tx.add(R.id.id_content, mFOne, "ONE");  
            tx.commit();  
        }  
          
    }  
  
}  
```

至于Fragment重启时如何保存数据？其实和Activity类似，Fragment也有onSaveInstanceState的方法，在此方法中进行保存数据，然后在onCreate或者onCreateView或者onActivityCreated进行恢复都可以。

# FragmentManager

FragmentManager栈视图
（1）每个Fragment以及宿主Activity(继承自FragmentActivity)都会在创建时，初始化一个FragmentManager对象，处理好Fragment嵌套问题的关键，就是理清这些不同阶级的栈视图。
下面给出一个简要的关系图：

![](http://upload-images.jianshu.io/upload_images/937851-6e0b034db7df7199.png?imageMogr2/auto-orient/strip%7CimageView2/2)

对于宿主Activity，getSupportFragmentManager()获取的FragmentActivity的FragmentManager对象;

对于Fragment，getFragmentManager()是获取的是父Fragment(如果没有，则是FragmentActivity)的FragmentManager对象，而getChildFragmentManager()是获取自己的FragmentManager对象。


# FragmentPagerAdapter与FragmentStatePagerAdapter
那么这两个类有何区别呢？
主要区别就在与对于fragment是否销毁，下面细说：

FragmentPagerAdapter：对于不再需要的fragment，选择调用detach方法，仅销毁视图，并不会销毁fragment实例。

FragmentStatePagerAdapter：会销毁不再需要的fragment，当当前事务提交以后，会彻底的将fragmeng从当前Activity的FragmentManager中移除，state标明，销毁时，会将其onSaveInstanceState(Bundle outState)中的bundle信息保存下来，当用户切换回来，可以通过该bundle恢复生成新的fragment，也就是说，你可以在onSaveInstanceState(Bundle outState)方法中保存一些数据，在onCreate中进行恢复创建。

如上所说，使用FragmentStatePagerAdapter当然更省内存，但是销毁新建也是需要时间的。一般情况下，如果你是制作主页面，就3、4个Tab，那么可以选择使用FragmentPagerAdapter，如果你是用于ViewPager展示数量特别多的条目时，那么建议使用FragmentStatePagerAdapter。


# 使用FragmentPagerAdapter+ViewPager的注意事项

1、使用FragmentPagerAdapter+ViewPager时，切换回上一个Fragment页面时（已经初始化完毕），不会回调任何生命周期方法以及onHiddenChanged()，只有setUserVisibleHint(boolean isVisibleToUser)会被回调，所以如果你想进行一些懒加载，需要在这里处理。

2、在给ViewPager绑定FragmentPagerAdapter时，new FragmentPagerAdapter(fragmentManager)的FragmentManager，一定要保证正确，如果ViewPager是Activity内的控件，则传递getSupportFragmentManager()，如果是Fragment的控件中，则应该传递getChildFragmentManager()。只要记住ViewPager内的Fragments是当前组件的子Fragment这个原则即可。

3、你不需要在“内存重启”的情况下，去恢复的Fragments，有FragmentPagerAdapter的存在，不需要你去做恢复工作。

# 封装BaseFragment基类

```java
public abstract class BaseFragment extends Fragment {
    protected View mRootView;

	@Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        if(null == mRootView){//判断一下你的mRootView是null再inflate，在ViewPager中随着页面滑动这个方法会调用多次，inflate过了之后就直接用就好了。
           mRootView = inflater.inflate(getLayoutId(), container, false);
        }
        return mRootView;
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        afterCreate(savedInstanceState);
    }

protected abstract int getLayoutId();

protected abstract void afterCreate(Bundle savedInstanceState);
}
```

为了实例化View,抽象一个getLayoutId方法，子类无需关心具体的创建操作，父类来做View的创建处理。同时可以提供一个afterCreate抽象函数，在初始化完成之后调用,子类可以做一些初始化的操作，你也可以添加一些常用的方法在基类，例如ShowToast().

# Fragment里监听虚拟按键和实体按键的返回事件
给rootView设置一个OnKeyListener来监听key事件
```java
mRootView.setFocusable(true);
mRootView.setFocusableInTouchMode(true);
mRootView.setOnKeyListener(new View.OnKeyListener() {
    @Override
    public boolean onKey(View v, int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            //不一定是要触发返回栈，可以做一些其他的事情，我只是举个栗子。
            getActivity().onBackPressed();
            return true;
        }
        return false;
    }
});
```

# 没有布局的Fragment的作用
没有布局文件Fragment实际上是为了保存，当Activity重启时，保存大量数据准备的

# 在Activity里面添加Fragment
建议这么写：
```java
public class MainActivity extends FragmentActivity  
{  
      
    private ContentFragment mContentFragment  ;   
  
    @Override  
    protected void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
      
        FragmentManager fm = getSupportFragmentManager();  
        mContentFragment = (ContentFragment) fm.findFragmentById(R.id.id_fragment_container);  
          
        if(mContentFragment == null )  
        {  
            mContentFragment = new ContentFragment();  
            fm.beginTransaction().add(R.id.id_fragment_container,mContentFragment).commit();  
        }  
  
    }  
  
}  
```
1、为什么需要判null呢？
主要是因为，当Activity因为配置发生改变（屏幕旋转）或者内存不足被系统杀死，造成重新创建时，我们的fragment会被保存下来，但是会创建新的FragmentManager，新的FragmentManager会首先会去获取保存下来的fragment队列，重建fragment队列，从而恢复之前的状态。  
2、add(R.id.id_fragment_container,mContentFragment)中的布局的id有何作用？
一方面呢，是告知FragmentManager，此fragment的位置；另一方面是此fragment的唯一标识；就像我们上面通过fm.findFragmentById(R.id.id_fragment_container)查找~~

# Fragment的startActivityForResult
（这里的两个Fragment是指在不同的Activity中的）
我们点击跳转到不同Activity的Fragment中，并且希望它能够返回参数，那么我们肯定是使用Fragment.startActivityForResult来start新的Fragment ; 
在Fragment中存在startActivityForResult（）以及onActivityResult（）方法，但是呢，没有setResult（）方法，用于设置返回的intent，这样我们就需要通过调用getActivity().setResult(ListTitleFragment.REQUEST_DETAIL, intent);。

可以看出：fragment能够从Activity中接收返回结果，但是其自设无法产生返回结果，只有Activity拥有返回结果。

# 同一个Activity的两个Fragment返回数据

可以用接口来实现  
也可以参考 <http://blog.csdn.net/lmj623565791/article/details/42628537>

# 使用建议

1. Activity向Fragment传递数据用new Instance(...)、setArguments(Bundle args)、getArugments()
2. 在BaseFragment中定义一个Activity的全局变量，在onAttach中初始化，在onDetch中释放
3. 如果你有一个很高的概率会再次使用当前的Fragment，建议使用show()，hide()，可以提高性能。当然如果你的app有大量图片，这时更好的方式可能是replace，配合你的图片框架在Fragment视图销毁时，回收其图片所占的内存。
4. 谨慎使用popStackBack(String tag/int id,int flasg)系列的方法


# 参考链接


<http://www.jianshu.com/p/d9143a92ad94>

<http://blog.csdn.net/lyhhj/article/details/51174973>

<http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0605/2996.html>

<https://www.zhihu.com/question/39662488/answer/82469372>

<http://www.jianshu.com/p/626229ca4dc2>