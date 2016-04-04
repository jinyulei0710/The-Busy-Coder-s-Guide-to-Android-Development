###有图片？

Android支持PNG,JPEG以及GIF的图片格式。GIF官方是不鼓励使用的，然而，是总的来说可取的格式。
Android也支持一些基于XML的专有图片格式，尽管在书的后面才会讨论到那个深度。

默认给这些所谓drawable资源的目录是res/drawable/。所有在那找到的图片可以从Java代码或其它地方(例如manifest)引用到，无关设备特性。

但是，你的存根目录并没有一个res/drawable／目录。

取而代之的，它有着像res/drawable－mdpi和res/drawable-hdpi这样的目录。

它们涉及到不同的资源集。后缀(例如，-mdpi,hdpi)是过滤器，指示在什么情形喜爱存储在这些目录中的图片应该被使用。
具体的，-ldpi指示图片指示图片应该在有着低密度的屏幕(大约每英寸120个像素点，dpi)的设置上使用。
-mdpi后缀指示资源适用于中等密度屏幕的(大约160dpi),-hdpi指示资源是用于高密度屏幕的(大约240dpi)。
-xhdpi指示资源是用于超高密度屏幕的(大约320dpi),-xxhdp指示用于超超高密度屏幕(大约480dpi),
-xxxhdpi指示用于超超超高密度屏幕(大约640dpi)，等等。


在每个这些目录中，你会看到一个ic_launcher.png文件(可能还有其它图标)。这是被你的应用用于显示在主界面应用启动器上的固有图标。
每个图片都是相同的图标，但是高密度的图标有着更多的像素。目标是在每个设别上图片看起来差不多同个尺寸，使用较高密度的有着
更详细的图片。

例如，我们的EmPubLite教程项目有着像res/drawable-hdpi／，res/drawable-xhdpi／，以及
res/drawable-mdpi/的目录。每个可以包含为密度定制的固有启动图标(ic_launcher.png)。

我们的AndroidManifest.xml文件然后在<application>元素中引用了我们的icon_launcher图标：

	<?xml version="1.0" encoding="utf-8"?>
	<manifest xmlns:android:"http://schemas.android.com/apk/res/android"
	 package="com.commonsware.empulite"
	 android:versionCode="1"
	 android:versionName="1.0">

	<supports-screens
	 android:largeScreens="true"
	 android:normalScrens="true"
	 android:smallScreens="false"
	 android:xlargeScreens="true"/>
	 <uses-sdk
	  android:minSdkVersion="15"
	  android:targetSdkVersion="18"/>

	  <application
	   android:allowBackup="false"
	   android:icon="@drawable/ic_launcher"
	   android:label="@string/app_name"
	   android:theme="@style/AppTheme">
	   <activity
	     android:name="EmpubLiteActivity"
	     android:label="@string/app_name">
	     <intent-filter>
           <action android:name="android.intent.action.MAIN"/>

           </category android:name="android.intent.category.LAUNCHER"/>
	     </intent-filter>
	     </activity>
	     </application>
	     </manifest>

注意，manifest仅仅引用了@drawable/icon_launcher,来告诉Android找到一个叫做
ic_launcher的drawable资源。资源文件引用并没有指示文件的资源类型-在资源标识中没有.png。
这就意味着ic_launcher.png和ic_launcher.jpg不能在同一个项目中，因为它们以相同的标识符
识别。你需要保持“基本名"(文件名没有扩展)在所有你的图片中是唯一的。

同样的，@drawable/ic_launcher引用并没有提及到使用什么屏幕尺寸。那是因为Android会基于运行你
应用的设备选择要使用的正确屏幕密度。你不需要明确担心从你的多个图标拷贝中去选。

####额....但我有的是“Mipmaps"?

如果你通过Android Studio的新项目向导创建了一个项目，结果是项目中不会有关于res/drawables-*／的目录，
而是mipmap目录(例如，res/mipmap-hdpi/)。

这几乎是没有文档化的。实际上，它们都是drawables，只不过是一个以@mipmap/..而不是@drawable／...引用。

对于大都数drawables来说，把它们放进res/drawable-*/目录，即使你不得不自己来创建它们。


####获取Android Drawables

你可能是一个平面设计师。或者，你可能认识一位平面设计师。在这些情形中，你可以
创建你自己的icons，理想情况下遵循Google的[图标设计指导](https://www.google.com/design/spec/style/icons.html)。

如果你不是一个平面设计师并且没有接触到一个，你需要以其它方式来获取你的drawable。有很多来自
第三方的图标库，但是以下章节略述了一些Google的用于把图标放进你应用的解决方案。


#####Android 图标设置向导

Android Studio和Eclipse都提供了一个图标设置向导。这个向导是为了获取一张启动图片然后给予你用于某个图片职责的多个密度图标而设计的，
例如你的启动图标(我们早前在这章看到的ic_launcher文件)。

在Eclipse上，图标设置向导会给你drawable资源。在AndroidStudio上，如果你选择创建启动图标的话，图标设置向导会给你
mipmap资源，如果你选择其它类型图标，它会给你drawable资源。

#####`Android Asset` 工作室

图标设置向导的功能可以存在于集成开发环境之外(在一个Chrome浏览器中)以[`Android Asset`工作室]的形式。有了
这个图标设置向导，你就可以选择一个图标类型了(例如，启动图标):

然后你可以指定基本图片的来源(上传的文件，剪贴画或自由形式的文本)和其它配置数据。产生的多个密度的图片，可以从页面底部进行下载。

#####编辑已经存在的drawable资源

不管Android Studio还是Eclipse都没有用于PNG和JPEG文件的图片编辑器。因此，你会在集成开发环境之外进行这些图片的编辑。






