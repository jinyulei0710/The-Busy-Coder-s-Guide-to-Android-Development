###配置常备的构建类型
在一系列可行的默认构建方式中，debug和release是你现成可以用的。但是，你可以改变这些默认值并做出这些构建方式工作方式的调整。这呢，我们会看下改变构建类型选项，关注的是常备的debug和release构建类型。

####源集(Sourceset)
每个构建类型有着它自己相关的源集。如果你跳过了这个给它的目录，那就意味着构建类型没有改变main源集。

所以，在Gradle／HelloConfig 样例项目中，在debug源集中我们会有一个strings.xml资源的替代版本：

	HelloCofig
	|-- build.gradle
	|-- HelloConfig.keystore
	|-- libs/
	|   |--android-support-v4.jar
	|-- local.properties
	|-- proguard-project.txt
	|-- project.properties
	|-- src/
    	|-- debug/
    	|   |--res/
    	|       |--values/
    	|           |--strings.xml
    	|--main/
        	|--AndroidMainfest.xml
        	|--assets/
        	|--java/
        	|   |--com/
        	|       |--commonsware
        	|           |--android/
        	|               |--gradle／
        	|                   |--hello/
	MainActivity.java
        	|--res/
            	｜--drawable-hdpi/
            	|   |--ic_launcher.png
            	|-- drawable-ldpi/
            	|-- drawable-mdpi/
            	|   |--ic_launcher.png
            	|-- drawable-xhdpi/
            	|   |--ic_laucher.png
            	|-- layout/
            	|   |--activity_main.xml
            	|--menu/
            	|   |--main.xml
            	|--values/
            	|   |--dimens.xml
            	|   |-strings.xml
            	|   |--styles.xml
            	|--values-sw600dp/
            	|   |--dimens.xml
            	|--values-sw720dp-land/
            	|   |--dimens.xml
            	|-- values-sw720dp-land/
            	|   |--dimens.xml
            	|-- values-v11/
            	|   |--styles.xml
            	|-- values-v14/
                	|--styles.xml           
                	
string.xml 包含了一个app_name的修改版，为了更明显显示我们运行的是这个应用的debug版本。

	<?xml version="1.0" encoding="utf-8"?>
	 <resources>
	 <string name="app_name">HelloGradle DEBUG</string>
	 </resources>
正如我们所看到的，构建类型源集中的资源替代了它们main中对应部分。或者，如果希望的话，构建类型可以添加新的main没有的新的资源。

####build.gradle设置
我们也可以使用build.gradle中的buildTypes闭包来配置debug和release构建类型的行为。在这个样例项目中，我们两个都改了，外加一些其他修改:

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
	}
	
正如在介绍给Android的Gradle中提到的，默认配置（defaultConfig）闭包允许我们改变AndroidManifest文件中发现的方面，取代main源集中的实际文件。

构建类型（buildTypes）闭包是我们定义构建类型行为的地方。每个构建类型在构建类型中自己的闭包中都能进行配置，并且我们能在那覆盖多种特性。

构建类型中我们能详细指明的值得注意的特性包含：

* debuggable(覆盖mainfest元素中的android：debuggable，指示应用是可调试的)
* applicationIdSuffix(在mainfest或默认配置中的application属性上加上后缀)
* versionNameSuffix（在mainfest或默认配置中的versionName属性上加上后缀）

(注意:applicationIdSuffix在给Android的Gradle的0.11release之前的版本是叫packageNameSuffix的)

我们的debug构建版本在version name和作为package name applicationId后面加上了后缀。注意修改applicationID只影响Android安装和运行时候的看到的包名。它并不影响R.class所处创建的目录，它用的是AndroidManifest.xml文件中的包名。它也不影响任何任何包中的类，就跟我们写的那时候一样。因此，大都数我们的代码对真实包名的改变是不在意的。例如在PackgeManger中查找东西或者构建组件名的时候，在任意Context（像活动）上使用，而不是一些硬编码的字符串，正如getPackageName()返回的是运行时环境认为的包名，这个包名会包含在构建过程添加的任意后缀。或者使用BuildCondig.APPLICATION_ID，在你没有上下文可以去调用getPackageName()的时候。

我们也可以有一个signingConfig属性，来配置我们的APK文件如何进行数字签名。这会在之后的章节涵盖到。	

####优先顺序
给构建类型定义的特性，给默认配置定义的特性会覆盖在Androidmanifest文件中的它们相对应的部分。但是，一个构建类型的源集也可以有它自己的AndroidManifest.xml文件。全部的优先顺序是：

* build.gradle中的优先处理
* 构建类型的AndroidManifest中的优先处理
* main源集的AndroidManifest中的

但是，合并清单总的来说是一个复杂的话题，会在之后单独一个章节进行讲解。

来自构建类型源集的资源会合并进main源集的资源，并且如果有冲突的话，构建类型资源优先。资产也是这样。

但是，Java源代码就有细微的区别了。构建类型的源集仍旧和main源集合并在一起，但是如果冲突的话，结果就是构建错误。构建类型源集和main源集两者之一可以定义任意的源文件，但是不是都可以。所以，当debug可以有一个版本的/you/p,ackage/name/Foo.java，release也能有一个不一样的your/package/name/Foo.java，main就不可也有your/package/name/Foo.java。因此，如果你在构建类型中定义了一个类，你可能需要在所有构建类型中定义这个类。那么任意来自main对这个类的引用对所有构建类型都是适用的。

但是对debug仅有的活动而言就不需要了。假定你想要你的应用中的一个活动提供给开发者缓存状态和其他内存结构的诊断信息。虽然你可以通过调试软件得到，但是有点烦人，仅仅轻触启动icon会更简单。但是你不想要，更不说用说需要，这个诊断活动存在在你的release构建中。为了做这项工作，你要把活动的Java类连同它的资源和manifest入口（包括Main/LAUNCHER<intent-filter>）一起只放进debug源集.由于main源集没有引用到你的诊断活动，release构建版本就不需要实现这个Java类。
	              	