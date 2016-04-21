###动作栏，生动的色彩

在Android 4.0上，如果以Holo主题作为基础，你可能想要调整你的动作栏所使用的色彩。

在Android 5.0+上，如果你使用原质主题作为基础，你会想要调整你的动作栏所使用的色彩。这是
谷歌希望的品牌设计运行的方式，替代掉你动作栏中的图标。

接下来的几节内容提出了一些影响你动作栏色彩的方法。

####原质着色效果

Android 5.0和Theme.Material使动作栏的色彩的设备变得简单，作为整个着色方案的一部分。

ActionBar/MaterialColor样例项目是早前这章展示的ActionBarDemoNative样例的一个拷贝，但是这里的这个：

* 我们的minSdkVersion被设为了21，所以应用只能运行在Android5.0+上。
* 我们使用特定的着色规则设置了一个自定义主题，这个主题会影响我们的动作栏颜色。

#####色彩资源

主题会需要引用色彩，并且最整齐的方式是设置色彩资源。像所有我们其它资源一样，我们给予色彩资源一个名称和一个色彩值，通常以#RRGGBB或#AARRGGBB的格式。色彩资源是“值”资源，默认由res/values/持有，
协定使用一个colors.xml文件用于真实的颜色。

例如，这里有个来自MaterialColor样例应用的res/values/colors.xml文件。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	  <color name="primary">#3f51b1</color>
	  <color name="primary_dark">#1a237e</color>
	  <color name="accent">#ffee58</color>
	 </resources>

它定义了三个颜色，primary,primary_dark,和accent,代表不同的颜色。在AndroidStudio中，编辑这个文件显示一个小小的颜色样品来帮助你想象这个颜色：

#####给主题着色

然后，假定我们已经定义了我们的颜色，我们可以应用这些颜色到一个自定义的主题上，存在于res/values/styles.xml：
	
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	  <style name="AppTheme" parent="android:Theme.Material">
	   <item name="android:colorPrimary">@color/primary</item>
	   <item name="android:colorPrimaryDark">@color/primary_dark</item>
	   <item name="android:colorAccent">@color/accent</item>
	  </style>
	 </resources>
	 
这里，我们的AppTheme是从Theme.Material继承而来的并且重载了三个色调：colorPrimary,
colorPrimaryDark,和colorAccent,转而指向我们的三个色彩资源。

注意如果我们想要一个光亮的内容区域，我们可以从Theme.Material.Light继承，或者使用Theme.Material.Light.DarkActionBar用于一个光亮的内容区域和一个深色的动作栏(在我们开始调整动作栏颜色之前)。

#####应用主题

应用的manifest声明了我们要使用的AppTheme作为我们<appliaction>的默认主题，那么所有的活动会使用这个主题，除非在活动层面进行重载：

	<?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
	package="com.commonsware.android.abmatcolor"
	android:versionCode="1"
	android:versionName="1.0">

	<supports-screens
		android:anyDensity="true"
		android:largeScreens="true"
		android:normalScreens="true"
		android:smallScreens="true"/>

	<uses-sdk
		android:minSdkVersion="21"
		android:targetSdkVersion="21"/>

	<application
		android:allowBackup="false"
		android:icon="@drawable/ic_launcher"
		android:label="@string/app_name"
                android:theme="@style/AppTheme">
		<activity
			android:name="ActionBarDemoActivity"
			android:label="@string/app_name">
			<intent-filter>
				<action android:name="android.intent.action.MAIN"/>

				<category android:name="android.intent.category.LAUNCHER"/>
			</intent-filter>
		</activity>
	</application>

    </manifest>

也注意到这么一点我们指定的minSdkVersion是21。Android Studio项目可以在build.gradle中去指定，但是因为Eclipse不能使用Gradle,我们必须把它定义在manifest中。

#####结果

包含我们填充的activity和ListView在内的所有其它东西都和之前的ActionBarDemoNative样例一样。

但是，当我们运行这个版本在Android 5.0+设备或模拟器上是，我们的动作栏披上了所要求的色彩，具体的就是colorPrimary值用于动作栏的背景色:

自定义主题，也会影响特定控件的颜色，就如之后内容涵盖到的那样。

#####恢复图标

当原质设计原则跳过了我们之前在动作栏中使用的应用图标，有一种方式来把它加回到Theme.Material应用，尽管这需要一点点的工作量，就如在ActionBar/MaterialLogo样例项目所看到的：

关键的事情是你需要在你的ActionBar对象上调用setDisplayShowHomeEnabled（true）,而这个对象是在你的Activity通过调用getActionBar()获取的：

    @Override
    public void onCreate(Bundle icicle) {
    super.onCreate(icicle);

    getActionBar().setDisplayShowHomeEnabled(true);

    initAdapter();
    }

在你的manifest中，不管什么图标设备给了android:icons属性，它都会使用这个图标作为你动作栏的“home”图标：

如果你要使用不同的图标，例如一个缩放的能更好适应动作栏的图标，你可以在你的
ActionBar上调用setIcon()方法，提供一个图标资源的ID(例如，R.drawable.action_bar_icon),这个图标会替代那个定义在manifest中你的<activity>或<application>的android:icon属性。







	 
	   	  