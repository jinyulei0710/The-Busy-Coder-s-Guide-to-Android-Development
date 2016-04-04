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

         
这里，<application>元素中的android:label属性引用了app_name字符串资源。这个会出现在应用程序的几个地方，
尤其是设置中的已安装程序列表。所以，如果你想要改变你应用名如何在这些地方显现，只要挑战app_name字符串资源就能满足。

@string／app_name语句告诉Android"找到叫做app_name的字符串资源"。这导致Android扫描合适的strings.xml文件(或你的res/values/目录下其它
包含字符串资源的文件)来尝试找到app_name。

####样式文本

许多Android中的东西可以显示富文本，其中文本使用一些轻量化的HTML标记:<b>,<i>和<u>进行了格式化处理。
你的字符串资源仅仅以你在网页中使用HTML标签的方式支持了这个。

      <resources>
        <string name="b">This has<b>bold</b>in it.</string>
        <string name="i">Whereas this has <i>italics</i>!</string>
      </resouces>


CDATA.CDATA 跑起来。跑起来，DATA,跑起来

因为字符串串资源文件是一个XML文件，如果你的信息中包好了<,>或&字符(以及上述列出的格式化标签)，
你会需要使用一个`CDATA section`:

    <string name="report_body">
    <![CDATA[
    <html>
    <body>
    <h1>TPS Report for :{{reportDate}}</h1>
    <p>Here are the contents of the TPS report:</p>
    <p>{{message}}</p> 
    <p>If you have any questions reguarding this report,please
    do <b>not</b> ask Mark Murphy.</p>
    </body>
    </html>
    ]]>
    </string>

####目录名

在我们存根项目中的字符串资源是位于res/values/strings.xml文件中的。因为这个目录名
没有后缀，在这个文件夹下的字符串资源在任何情形下都是有效的，包含所有的设备地点。
我们会需要附加的有着不同strings.xml文件的目录，来支持其它语言。我们会在这[本书之后内容]()涵盖到如何去做.

####编辑文件资源

如果你在一个字符串资源文件上双击，像res/values/strings.xml。在AndroidStudio中，XML会呈现在出来然后就能修改了。






