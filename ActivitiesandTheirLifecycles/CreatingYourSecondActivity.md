###创建你的第二个(以及第三个以及...)Activity

不幸的是，activity们并不会自己创建。从积极面看，这能帮助Android开发者维持被高薪聘请。

因为，在只有一个activity的项目之上，如果你想要第二个activity，你需要自己添加它。
第三个，第四个也是这样，以此类推。

这节分析的样例是Activities/Explicit。我们的首个Activity,ExplictIntentsDemoActivity,
由构建工具生成的默认activity启动。尽管现在，它的布局只包含了一个按钮:

	<?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical">

	<Button
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:textSize="20sp"
		android:text="@string/hello"
		android:onClick="showOther"/>

    </LinearLayout>
    
这个Button是绑定到我们activity实现的一个showOther()上的，我们会简短地进行分析这个方法。

####定义类和资源

为了创建你的第二个(或第三个或不管第几个)的activity，你首先需要创建Java类。
你需要创建一个新的Java源文件，其中包含一个直接或间接继承Activity的公有Java类。你有两种实现方法：

* 自己创建类和资源
* 使用AndroidStudio的新建activity向导

#####AndroidStudio和新建Activty向导

在你的项目浏览器的app/src/main源集上右击，然后到上下文菜单的New>Activity部分。这会给你一个可用
activity模板的子菜单-大部分和我们最初创建项目看见的是一样的。

如果你选择了这些模板中的一个，一个提供这个activity详情的向导页面会出现：

这里所看到的是由你选择的模板所决定的(例如，activity名称，布局XML资源名称)并且是和新建向导中我们所看到的
是类似的。

点击“Finish”然后会创建activity的Java类，相关的资源，以及manifest入口。

####填充类和资源

一旦你的存根activity创建好了，你就可以在其中添加一个onCreate()(或者通过向导编辑一个已经存在的activity)，
填充所有的细节(例如，setContentView()),就和你在首个activity做的一样。你新建的activity可能需要一个新的布局XML资源或其它的资源，这些也是要你自己创建（或由向导为你创建的）。

在Activities/Explict中，我们的第二个activity是OtherActivity,有着相当标准的
基础实现：

	package com.commonsware.android.exint;
	
	import android.app.Activity;
	import android.os.Bundle;
	
	public class OtherActivity extends Activity{
	   @Override
	   public void onCreate(Bundle savedInstanceState){
	       super.onCreate(savedInstanceState);
	       setContentView(R.layout.other);
	   }
	}


以及类似的简单布局,res/layout.other.xml:

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical">

	<TextView
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:text="@string/other"
		android:textColor="#FFFF0000"
		android:textSize="20sp"/>

    </LinearLayout>
    
#####向Manifest中加东西

仅仅只有一个activity实现是不够的。我们也需要在我们的AndroidManifest.xml文件中添加它。
这是由集成开发环境的的各自的新建activity向导自动为你处理的。但是如果你自己手动创建的activity的话，你会需要添加它的manifest元素，久而久之你会需要在很多情形下编辑这个元素。


添加一个activity到manifest就是添加另一个<activity>元素到<application>元素中的问题:

    <?xml version="1.0" encoding="utf-8"?>
     <manifest xmlns:android="http://schemas.android.com/apk/res/android"
	 package="com.commonsware.android.exint"
	 android:versionCode="1"
	 android:versionName="1.0">

	<uses-sdk
		android:minSdkVersion="7"
		android:targetSdkVersion="11"/>

	<supports-screens
		android:largeScreens="true"
		android:normalScreens="true"
		android:smallScreens="true"/>

	<application
		android:icon="@drawable/ic_launcher"
		android:label="@string/app_name">
		<activity
			android:name="ExplicitIntentsDemoActivity"
			android:label="@string/app_name">
			<intent-filter>
				<action android:name="android.intent.action.MAIN"/>

				<category android:name="android.intent.category.LAUNCHER"/>
			</intent-filter>
		</activity>
		<activity android:name="OtherActivity"/>
	</application>

    </manifest>  
    
你至少需要android:name属性。注意到我们并没有像最初activity那样，包含一个<intent-filter>子元素。
对于目前来说，相信最初activity的<intent-filter>是导致一个可启动的activity出现在主界面launcher的原因。
当之后这章，你可能想要自己的<intent-filter>的时候，我们会深入更多关于<intent-filter>如何工作的细节。  
    

    









    
    

