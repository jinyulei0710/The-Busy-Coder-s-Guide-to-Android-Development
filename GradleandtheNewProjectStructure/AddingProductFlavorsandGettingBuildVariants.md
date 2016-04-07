###添加产品风格获取构建变种
许多应用不需要产品风格，但是一些需要。在很多方面,添加一个产品风格是类似的，为了添加一个构建类型，正如我们将在Gradle/HelloProductFlacors样例中看到的，它基于之前的样例添加了一对产品风格：vanilla和chocolate。 （注意：产品风格不需要在真正的产品风格之后命名）

每个产品风格和每个构建类型一样，能有它自己的源集。在此例中，我们有的是代表各自代码集的src/vanilla和src/chocolate/目录:

	HelloProductFlavors
	|-- build.gradle
	|-- HelloConfig.keystore
	|-- libs/
	|   |-- android-support-v4.jar
	|-- local.properties
	|--  proguard-project
	|-- src/
    	|--chocolate/
    	|   |--java/
    	|       |--com/
    	|           |--commonsware
    	|               |--android/
    	|                   |--gradle
    	|                       |--hello/
    	|                               |--
	MainActivityOptionsStrategy.java
    	|-- debug/
    	|   |--res/
    	|       |--values/
    	|           |--strings.xml
    	|--main/
    	|   |--AndroidManifest.xml
    	|   |--assets/
    	|   |--java/
    	|   |   |--com/
    	|   |       |--commonsware
    	|   |           |--android/
    	|   |               |--gradle
    	|   |                   |--hello
    	|   |
	MainActivity
    	|   |--res/
    	|       |--drawable-hdpi/
    	|       |   |--ic_launcher.png
    	|       |--drawable-ldpi/
    	|       |--drawable-mdpi/
    	|       |   |--ic_launcher.png
    	|       |--drawable-xhdpi
    	|       |   |--ic_launcher.png
    	|       |--layout/
    	|       |   |--activity_main.xml
    	|       |--menu/
    	|       |   |--main.xml
    	|       |-- values/
    	|       |   |--dimens.xml
    	|       |   |--strings.xml
    	|       |   |--styles.xml
    	|       |--values-sw600dp/
    	|       |   |--dimens.xml
    	|       |--values-sw720dp-land/
    	|       |   |--dimens.xml
    	|       |--values-v11/
    	|       |   |--styles.xml
    	|       |--values-v14/
    	|           |--styles.xml
    	|-- vanilla/
        	|--java/
            	|--com/
                	|--android/
                    	|--gradle/  
                        	|--hello/
                            	|--
	MainActivityOptionStrategy.java

在源集中，我们有一个MainActivityOptionsStrategy类，每个产品风格都实现了一个。这个类被main源集中的MainAcitivity新的实现所引用，来代理处理onOptionsItemSelected():

	package com.commonsware.android.gradle.hello;

	import android.os.Bundle;
	import android.app.Activity;
	import android.view.Menu;
	import android.view.MenuItem;

	public class MainActivity extends Activity{
 	 @Override
  	protected void onCreate(Bundle savedInstanceState){
  	super.onCreate(savedInstanceState);
  	setContentView(R.layout.activity_main);
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu){
 	getMenuInfalter().inflate(R.menu.main,menu);
 	return true;
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item){
   	return(MainActivityOptionsStrategy.onOptionsItemSelected(item))
	}
	
因为我们不需要在menu选项中没有太多的事情要做，每个产品风格的实现MainActivityStrategy只是打印风格特性消息到LogCat，例如如下vanilla的:

	package com.commonsware.android.gradle.hello;

	import android.util.Log;
	import android.view.MenuItem;

	public class MainActivityOptionsStrategy{
   	public static boolean onOptionsItemSelected(Menu item){
      	Log.d("HelloProductFlavors",vanilla!);
   	}
	}

告诉Android关于我的的产品风格，并配置它们的行为，在build.gradle文件的我们有了一个新的productFlavors闭包：

	buildscript{
   	repositories{
     	mavenCentral()
   	}
   	dependencies{
     	class 'com.android.tools.build:gradle:1.0.0'
   	}
	}

	apply plugin: 'com.android.application'

	dependencies{
	}

	android{
	compileSdkVersion 19
	buildToolVersion "21.1.2"

	defaultConfig{
   	versionCode 2
   	versionName "1.1"
   	minSdkVersion 14
   	targetSdkVersion 18
	}

	signingCofigs{
     	release{
       	storeFile file("HelloCofig.keystore")
       	keyAlias "HelloCofig"
       	storePassword 'laser.yams.heady.testy'
       	keyPassword 'fw.stabs.steady.wool'
     	}
	}

	buildTypes{
     	debug{
       	applicationIdSuffix ".d"
       	versionNameSuffix "-debug"
     	}
     	release{
       	siningConfig signingConfigs.release
     	}
     	mezzanine.initWith(buildTypes.release)
     	mezzanine{
          	applicationIdSuffix ".mezz"
          	debuggable true
     	}
	}

	productFlavors{
    	vanilla{
       	applicationId "com.commonsware.android.gradle.hello.vanilla"
    	}
    	chocolate{
       	applicationId "com.commonsware.android.gradle.hello.chocolate"
    	}
	}
defaultConfig是使用被产品风格所使用的同样的对象类型实现的。因此，我们可以在产品风格中定义与defaultConfig中同样的东西，例如applicationId,正如这个build.gradle文件中做的一样。

就优先顺序而言

* 产品风格覆盖main源集和defaultConfig
* 构建类型覆盖产品风格
所以，vanilla产品风格的debug构建版本会生成的包名为com.commonware.android.gradle.hello.vanilla.d.

我们的任务的名字数量就多了而且更加复杂了，表达出产品风格和构建类型的差积。现在，除了installDebug,installMezzanine以及installRelease之外，我们有了：

 * installChocolateDebug 
 * installChocolateMezzanine 
 * installChocolateRelease
 * installVanillaDebug 
 * installVanillaMezzanine 
 * installVanillaRelease	

