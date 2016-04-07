###添加构建类型
对很多开发者而言，加上之前展示的一些调整，debug和release构建类型就够用了。尽管如此，少数开发者会有需要新类型的其它情景。幸运的是，添加一个新的类型是相当简单的，就如在Gradle/HelloBuildType样例项目中看到的，基于之前的样例中添加一个新的构建类型。

如同内置的构建类型一样，通过在src/目录下添加适当的目录，你的新的构建类型也能有它们自己的源集。并且，如同内置的构建类型一样，你可以在build.gradle的buildTypes闭包中配置新的构建类型。

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
	
在这个项目中，我们要第三个叫作mezzanine的构建类型，代表处于debug构建和release构建之前的中间地带。

为了告诉Android新的构建类型，我们需要初始化一个。这个mezzanine.initWith(buildTypes. release)声明进行处理的，它是在已经定义好的release构建版本之上初始化新的mezzanine构建版本。从那里开始，随后的mezzanine闭包就可以修改构建类型的属性了。在此例中我们：

* 在包名之后加上了一个.mezz后缀
* 标记项目为可调试
* 使用release的签名秘钥
因为我们是以release构建的配置开始的，实际结果就是我们有了一个等同于release构建的构建，仅仅是设置了下debuggable标记。

现在，我们得到了带着Mezzagnie的Gradle任务，像installMezzagnie,和Debug和Relase对应部分一起。	
	