###Manifest的剩余部分
不是每个manifest都可以在Gradle构建文件中重写。这里就有一些只会出现在mainfest中的关键项，不管是以AndroidStudio,Eclipse还是其它方式创建的.

####给应用的Application标签
在你的初始项目manifest中，元素的主要子元素是一个元素。

默认时，当你创建一个新的Android项目，你得到了处于中的单个元素：


	<?xml version=“1.0” >
	<manifest xml:android="https://schemas.android/apk/res/android"
 	package="com.commonsware.android.skeleton"
 	android:versionCode="1"
 	android:versionName="1.0">
 	<application>
 	<activity
 	android:name="Now"
 	android:label="Now">
 	<intent-filter>
 	<action android:name="android.intent.action.MAIN"/ >
	<category android:name="android.intent.category.LAUNCHE"/>
 	</intent-flter>
 	</ativity>
 	</aplication>
 	</manifest>

这个元素为实现活动的类提供了android:name，为活动名的展示提供了android:label，并且有时候提供一个<intent-filter>这个子元素来描述在什么条件下活动会被展示出来。常备的元素吧你的活动设置成启动页，所以用户可以选择运行它。正如我们将在之后这本书所看到的，如果你要选的话，你可以在一个项目中有许多个活动。

在此例中的android:name 属性，有一个光秃秃的Java类名(Now).有时候，你可以看到有着完全合格类命的android:name(例如,com.commonsware.android.skeleton.Now)。有时候，你会看到有着单个句号前缀的类命。Now和.Now都指向在你的项目包（你声明在中的package属性）中的Java类。

支持多样的屏幕
Android设置有着宽阔的屏幕范围，从2.0寸的小型智能手机到46寸的电视机。Android基于物理尺寸和通常间隔把这些分成四个桶：

1.小（3寸以下）

2.常规(3寸到4.5寸)

3.大（4.5寸到10寸）

4.超大(10寸以上)

默认情况下，你的应用会支持小的和常规的屏幕。它也会通过一些Android的自动转化进而支持大屏和超大屏。

为了真正支持所有你要的屏幕尺寸，你应该考虑在你的manifest中添加<supports-screen>元素。这个例举了你已经明确支持的屏幕尺寸。例如，你为大屏和或超大屏提供自定义UI支持，你将会需要<supports-screens>元素。所以，虽然初始的manifest能工作，处理多屏幕尺寸是你要考虑的问题。

即一个元素类似于:

	<supports-screens
	android:largeScreen="true"
	android:normalScreen="true"
	android:smallScreen="false"
	android:xlargeScreen="true”/>

关于支持所有屏幕尺寸的更多信息，包括<supports-screens>元素，会在涵盖大屏策略的时候讲到。

####其它东西

随着本书的进程，你会发现其它添加到manifest中的元素，例如

* <uses-permission>，告诉用户你需要权限去使用某一设备功能，例如访问网络
* <uses-feature>，告诉Android你需要设备有某一特性(例如,相机)，并且因此你的应用不能安装在缺少此属性的设备上。
* <meta-data>,被Android特定扩展需要的小块信息，例如Google Play Services 类库
这些以及其它与元素会在书中其它地方介绍。

