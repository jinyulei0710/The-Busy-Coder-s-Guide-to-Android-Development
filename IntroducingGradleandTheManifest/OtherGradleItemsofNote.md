###其它需要注意的Gradle事项

在android闭包中你直接至少有两个声明:compileSdkVersion和buildToolsVersion.

	android{
   	compileSdkVersion 19
   	buildToolsVersion "21.1.2"
	}

compileSdkVersion定义了编译的API等级，通常是简单的API等级整数（例如,19）。一个Eclipse项目则会把这个放在项目根目录的project.properoties文件中。

buidlToolsVersion指示了想要项目使用的AndroidSDK构建工具的版本。当从Maven中心下载的时候给了我们部分所需的，但不是完整的。其余部分来自SDK管理器的"AndroidSDK构建工具"入口。

意sdk管理器会允许你下载最新版本的Gradle构建工具（如上截图是20）以及之前的版本(例如.如上截图是18.0.1和17)。这和你的build.gradle文件中的buildToolsVersion相对应。

所以，你的android闭包会看起来像这样:
	android{
  	 compileSdkVersion 19
  	 buildToolsVersion "20.0.0"
   	defaultConfig{
     	 applicationId "com.commonsware.empulite"
     	 versioncode 1
      	versionName "1.0"
      	minSdkVersion 15
      	targetSdkVersion 18
   	}
	}

Eclipse并没有课配置构建工具版本概念，所以在一个Eclipse项目中就没有类似buildToolsVersion的东西。