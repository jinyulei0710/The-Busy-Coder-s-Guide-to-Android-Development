###Manifest和Gradle两者间共同的事项

有一些关键项可以被定义在manifest中的也可以在build.gradle声明中重写。这些项对于开发和操作我的Android应用都是相当重要的。

####包名和应用ID
所有manifest文件的根，毫无疑问，是一个manifest元素:


	<manifest xmlns:android:"https://schemas.android.com/apks/res/android"
	package="com.commonware.empulite">

注意android命名空间声明。你只能在许多属性上使用命名空间而不是元素上(例如，而不是<andorid:manifest>)

你需要提供给文件的信息的最大部分是包名属性。

包名属性一直都需要在manifest中，即使对AndroidStudio项目来说。包名属性会控制一些源代码在哪为我们生成，尤其是我们会在稍后遇到的一些R和BuildConfig类。

因为包名是用于生成Java代码的，它必须是合法的Java包名。java习惯包名应该基于一个颠倒的域名(例如,com.commonware.empulite),讨论的是你的域名。这样的话，重名就不太可能了。

包名也作为我们应用的默认"应用标识"。这需要一个唯一的标识，例如： 同一设备同一时间不能安装相同应用标识的两个应用
相同应用标识的两个应用不能上传到Google应用市场
默认情况，应用ID就是包名，但是AndroidStudio用户可以在他们的Gradle构建文件中进行覆盖。特别的，在android闭包中可以是默认配置闭包,并且在其中可以有一个applicationId声明：


	android{
  	 //other stuff
   		defaultConfig{
     	 applicationId "com.commonsware.empulite"
      	//more other stuff
   	}
	}

不仅AndroidStudio用户可以在默认配置闭包中覆盖应用标识，也有不同情形性爱有着不同应用标识的方式，例如调试构建对比发布构建。我们会在本书之后稍后内容中进行探讨。

####minSdkVersion和targetSdkVersion
你的manifest可能也包含了一个作为元素子元素的<uses-sdk>元素，来指定你支持什么版本的Android。此外，它能包含andorid:minSdkVersion和android:targetSdkVersion属性。Eclipse项目会一直有这个属性。AndroidStudio项目可能没有这个属性，因为这些值也能在默认配置闭包（应用标识可以被定义的地方）中定义为minSdkVersion和targetSdkVersion属性。

两个之中，更为关键的是minSdkVersion。它指示了你所测试应用所用的最老版本的Android。属性的值是代码AndroidAPI等级的整数。所以，你只在Android4.1或以上测试你的设备，你应该把你的minSdkVersion设置成16。

你也可以指定一个targetSdkVersion。这指示了你些代码时所考虑的Android版本。如果你的应用是运行在一个较新版本的Android上，Android可能作一些事来提高你代码的兼容性于较新Android相适应。现如今，大都数Android开发者应该制定目标SDK版本为15或者更高。随着本书内容的深入我们会开始探索更多关于targetSdkVersion的东西，但这时候，不管你的集成开发环境给你的默认值是多少，都可能是好的开始。

build.gradle中的对应条目就到默认配置闭包中了:


	android{
    	defaultConfig{
     	   applicatinId "com.commonsware.empulite"
       	 minSdkVersion 15
        	targetSdkVersion 19
       	 //more other stuff
    		}
	}

####版本号和版本名
在根元素之上，你的manifest清单也可以定义android:versionName和android:verisonCode属性。即使如此，一个AndroidStudio项目经常跳过这些，并在默认配置闭包中通过versionName和versionCode属性进行设置。

这两个值代表了你应用的版本。versionName的值是用户会在它们的应用设置里的应用详情界面会看到的一个版本指示。


同时，版本名称也被google应用市场目录所使用，如果你按此中方式发布你的应用的话。版本名称可以是任何你想要的字符串值。

另一方面，版本号一定要是一个整数值，并且一个新的版本必须比旧的版本的高的版本号。Andorid和Google市场会比较新的APK和已安装应用的版本号来决定是否要更新APK。通常的做法是版本号从1开始，并且在每次发布的时候增加它。当然，你可以选择其它你想要的约束。在开发期间，你可以把这些放在一边，但是当你产品化的时候，这些属性就变得很重要了。