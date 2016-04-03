###字符串理论

把你的标签和其它文本保存在你应用的main源代码之外，通常被认为是一个好主意。特别地，它对于国际化(I18N)和本地化(L10N)是有帮助的。
即使你不会要吧你的字符串翻译成其它语言，如果所有的字符串在一个地方比分布在你的源代码中改起来会比较容易。

####明文字符串

通常来讲，所有你需要的是在res/values目录中放一个XML文件(通常命名成res/values/strings.xml)，其中有一个resource根元素以及
一个子string元素用于你想要编码成资源的每个字符串。string元素有一个name属性，这个属性是这个字符串的唯一标识符，以及单个包含字符串文本的
text元素：

	<resources>
		<string name="quick">The quickbrown fox...</string>
		<string name="laughs">He who laughs lat...</string>
    </resources>

一个棘手的部分是如果字符串的值包含了一个引号或一个所有格符号。在这些情形中，你会想要以在它们之前加反斜线来转义这些值  
(例如，There are the times that try men\'s souls)。或者，如果它仅仅是个所有格符号，你可以吧这个值装入引号
（例如，“These are the times that try men's souls.")。

例如，一个项目的strings.xml文件可以像这个一样:

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
       
       <string name="app_name">EmpubLite</string>
       <string name="hello_world">Hello world!</string>

	</resources>

我们可以从我们Java源代码和其它多个地方引用这些字符串资源。例如，app_name字符串资源经常在AndroidManifest.xml文件中被使用:

	<?xml version="1.0" encoding="utf-8"?>
	<manifest
	 xmlns:android:"http://shcemas.android.com/apk/res/android"
     package="com.commonsware.empublite"
     android:versionCode="1"
     android:versionName="1.0"
     <application
       android:allowBackup="false"
       android:icon="@drable/ic_launcher"
       android:label="@string/app_name"
       android:theme="@style/AppTheme">
       <activity
         android:name="EmpuLiteActivity"
         android:label="@string/app_name"
         <intent-filter>
         <action
         android:name="android.intent.action.MAIN"/>
         <category
         android:name="android.intent.category.LAUNCHER"/>
         </intent-filter>
        </activity>
        </application>

        </manifest>

         








