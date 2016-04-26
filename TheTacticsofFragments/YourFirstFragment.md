###你的首个碎片

在多方面来说，看着一个实现方案来分析碎片比以抽象概念进行讨论简单一些。所以，在这一节中，
我们会看下Fragments/Static这个样例项目。这个项目几乎是来自前一章Acticities/Lifecycle样例项目的一个近乎克隆的项目。但是，我们把启动activity从自己本身直接持有控件到持有一个碎片，这个碎片转而进行控件管理。

####碎片布局

我们的碎片要管理我们的UI，所以我们有一个包含我们按钮的res/layout/mainfrag.xml布局文件：

	<?xml version="1.0" encoding="utf-8"?>
	<Button xmlns:android="http://schemas.android.com/apk/res/android"
	 android:id="@+id/showOther"
	 android:layout_width="match_parent"
	 android:layout_height="match_parent"
	 android:text="@string/hello"
	 android:textSize="20sp"/>
	 
注意，尽管如此，我们没有使用android:onClick属性。我们很快就会解释为什么我们丢掉来自前个版本的样例的这个属性。


####Fragment类

项目有一个会使用这个布局和处理按钮的ContentFragment类。

和activity一样，代表性的Fragment子类没有构造器。你要重载的主要方法，不是onCreate()。作为替代的，主要要
重载的方法是onCreateView()，它负责返回展示这个fragment的UI:

     View.OnClickListener {
     @Override
     public View onCreateView(LayoutInflater inflater,
                           ViewGroup container,
                           Bundle savedInstanceState) {
    View result=inflater.inflate(R.layout.mainfrag, container, false);

    result.findViewById(R.id.showOther).setOnClickListener(this);

    return(result);
    }
    
    
我们传入了一个我们可以inflate布局文件的LayoutInflater,ViewGroup最终会持有所有我们
inflate的东西，以及我们传入activity的onCreate()的Bundle.尽管我们习惯于用框架类为我们加载
我们的资源，我们也可以在任何时候使用LayoutInfalter infalte一个布局资源。这个过程中，读入XML，对它进行
解析，然后遍历元素树，为每个元素创建Java对象，并把结果融入到一个父子关系中。

这里，我们inflate了res/layout/mainfrag.xml，告诉Android它的内容最终会到ViewGroup中而不是直接添加。
尽管LayoutInflater上有更为简单风格inflate()方法，在此例中的ViewGroup恰好是相对布局，所以我们能够适当地处理所有的位置和尺寸规则。

我们也使用了findViewById()去找到我们的按钮控件并告诉它fragment是它的点击事件的监听者。ContentFragment必须要实现View.OnClickListener接口才能生效。作为android:onClick的替代方案，发送点击事件给fragment而不是activity。

因为我们实现了View.Onclicklistener 接口，我们需要对应的onClick()方法的实现:

	@Override
	public void onClick(View v){
	  (StaticFragmentDemoActivity)getActivity()).showOther(v);
	}
	
任何碎片都能调用getActivity()找出持有它的activity。在我们的示例中，唯一能可能持有这个碎片的activity是
StaticFragmentsDemoActivity，那么我们就能把getActicity()获得的结果强制转换成StaticFragmentsDemoActivity,那样的话我们就能在我们的activity上调用方法。尤其是，我们通过调用我们在最初Acticities/Lifecycle样例中看到的showOther()方法的方式，来告诉activity来展现另一个activity。

这是实际上这个碎片所有必须有的东西。但是，ContentFragment也可以重载许多其它fragment生命周期方法，并且我们稍后会在这章中分析这些方法。

####Activity布局

被activity所使用的res/layout/main.xml最初是用来放我们的按钮控件的。现在，按钮是由fragment处理的。
取而代之的，我们的activity布局需要容纳fragment本身。

在这个样例中，我们要使用的是一个静态fragment。静态fragment很容易添加到你的应用上，仅仅在布局文件中使用了
<fragment>元素，例如我们修改过的res/layout/main.xml文件:

    <?xml version="1.0" encoding="utf-8"?>
	<fragment xmlns:android:"http://shemas.android.com/apk/res/android"
	 android:layout_width="match_parent"
	 android:layout_height="match_parent"
	 android:name="com.commonsware.android.sfrag.ContentFragment"/>
	 
这里，我们把我们的UI声明称完全由一个fragment组成，它的实现(com.commonsware.android.sfrag.ContentFragment)是由<fragment>元素上的android:name属性所标识的。
作为android:name的替代方案，你可以使用class,尽管大都数的文档已经切换到了android:name。

如果需要的话，Android Studio用户可以从图形布局编辑器工具面板拖出一个fragment，而不是直接在XML中设置
<fragment>元素。

####Activity类

StaticFragmentsDemoActivity-我们新启动程序activity-和之前版本除了类名之外完全一模一样。

	package com.commonsware.android.sfrag;
	
	import android.content.Intent;
	import android.os.Bundle;
	import android.view.View;
	
	public class StaticFragmentDemoActivity extends 
	LifecycleLoggingActivity{
	  @Override
	  public void onCreate(Bundle savedInstanceState){
	     super.onCreate(savedInstanceState);
	     setContentView(R.layout.main);
	  }
	  
	  public void showOther(View v){
	     Intent other=new Intent(this,OtherActivity.class);
	     
	     other.putExtra(OtherActivity.EXTRA_MESSAGE,
	         getString(R.string.other));
	      startActivity(other);   
	  }
	
	}

因为res/layout/main.xml文件有了<fragment>元素，fragment仅仅在setContentView()被调用的时候就被加载到了
适当的位置上。




	 
	

	

