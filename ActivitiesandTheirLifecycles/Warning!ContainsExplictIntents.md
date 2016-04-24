###警告!包含显式意图!

一个意图封装了一个对Android作出的请求，用于让一些activity货其他receiver做一些事情。

如果你想要启动的activity是你自己的，你可能找到的最简单的方式就是创建一个显式的意图，命名你想要启动
的组件。例如，你可以在你的activity里创建一个这样的意图:

	new Intent(this,HelpActivity.class);

你会规定你想要启动的是HelpActivity.HelpActivity会需要在你的AndroidManifest.xml中有一个对应的
<activity>元素。

在Activities/Explict,ExplicitIntentDemoActivity有着一个绑定到按钮控件的onclick属性的showOther()方法。
这个方法会使用带有识别OtherActivity的显式意图的startActivity()：

	package com.commonsware.android.exint;

    import android.app.Activity;
    import android.content.Intent;
    import android.os.Bundle;
    import android.view.View;

    public class ExplicitIntentsDemoActivity extends Activity {
     @Override
     public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
     setContentView(R.layout.main);
    }
  
     public void showOther(View v) {
      startActivity(new Intent(this, OtherActivity.class));
     }
    }	
	
我们启动的activity显示了button:

点击按钮带出了另一个activity：

点击BACK会带我们回到第一个activity。在这一方面，Andorid中的BACK按钮就和你浏览器
中的返回按钮差不多。


	
	