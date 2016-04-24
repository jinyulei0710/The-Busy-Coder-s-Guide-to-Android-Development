###使用隐式意图

显式Intent方式在启动你自己的activity的时候是很好工作的。

但是，你可能会启动来自操作系统或第三方应用的activity。在这些情形中，在你的项目中你没有代表其它activity的Java类，那么你就不能使用以类作为参数意图构造器。

取而代之的，你会使用"隐式"意图结构，它看起来就像网页的工作方式一样令人讨厌。

如果你做过任何Web应用的工作的话，你会意识到HTTP是基于附加到URI的动词的：

* 我们想要GET这张图片
* 我们想要POST这个脚本到控制器
* 我们想要PUT这个倒REST资源
* 等等

Android的隐式意图模式差不多是相同的方式，只不过有更多的动词。


例如，你从某个地方获取了经纬度(例如，微博的内容，短信消息的内容)。你决定你要在地图上展示这些坐标。你有很多方式可以直接嵌入Google地图到你的应用，并且会在之后的章回看到底怎么做-但是展示复杂的并且是以用户想要使用Google地图为前提的。如果我们可以创建通用类型的请求，“你好，Android，展示一个显示这个坐标到地图上的activity”。

或者，在一个更为简单的情形中，我们获取到了来自某个来源的网页URL（例如，Web服务调用）并且我们想要在这页上代开一个浏览器。这会在Actities/LauncheWeb样例项目中进行举例说明。

其中，我们有一个使用了一个包含一个EditText控件和Button的布局的LauchDemo Activity。


    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical">

	<EditText
		android:id="@+id/url"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:hint="@string/url"
		android:inputType="textUri"/>

	<Button
		android:id="@+id/browse"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:onClick="showMe"
		android:text="@string/show_me"/>

    </LinearLayout>
    
    
按钮被绑定到了activity本身上的一个showMe()方法，在这个方法中我们带出了浏览一个输入到EditText的URL的浏览器。    

    package com.commonsware.android.activities;

    import android.app.Activity;
    import android.content.Intent;
    import android.net.Uri;
    import android.os.Bundle;
    import android.view.View;
    import android.widget.EditText;

    public class LaunchDemo extends Activity {
     @Override
      public void onCreate(Bundle icicle) {
       super.onCreate(icicle);
        setContentView(R.layout.main);
     }

     public void showMe(View v) {
        EditText url=(EditText)findViewById(R.id.url);

       startActivity(new Intent(Intent.ACTION_VIEW,
                             Uri.parse(url.getText().toString())));
     }
    }


这里，我们取得了URL并通过调用Uri.parse把它转换成Uri。然后，我们可以使用一个叫做ACTION_VIEW的动作去尝试展示所要的网页。


当启动的时候，展现在用户面前的是我们的数据入口表单：

我们可以填入一个URL：

如果设备上有以https：模式响应ACTION_VIEW意图的应用，点击“ShowMe！”按钮之后，会带出那个应用，很可能是一个网页浏览器：

在之后章回中，我们将会讨论如果没有处理这个意图的情况发生，或有多个应用可以进行处理的情况。





