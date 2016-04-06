###分析Gradle文件
一个AndoridStudio项目可能有两个build.gradle文件，一个是项目层面的一个是模块层面的（例如在，app/目录）。对于单模块项目来说分开是不需要的－只需要一个build.gradle文件就可以了。

不管你的build.gradle文件是在一个地方，你还是能找到一些功能元素的。这节会对其进行概述。

####构建脚本
Gralde中的构建脚本闭包（即，包含在括号中的代码块）是你配置JAR包和Gradle用来解释其它文件的地方。因此，你在配置构建脚本的时候你没有对项目进行配置。


	buildscript｛
 	  repositories{
      	mavenCentral
  	 }
   	dependencies｛
   	  class 'com.android.tools.build.gralde.1.0.0'
   	｝
	}

构建脚本中的仓库闭包指示了依赖的来源,通常是Maven样式的仓库。这里的话，mavenCenteal（）是一个内置的来设置Maven中心仓库信息的内置方法，一个流行的获取开源依赖的地方。

依赖闭包指示了要运行其余构建脚本需要的东西。classpath'com.android.tools.build:gradle:1.0.0‘，Gradle小组的文档不是很好。但是'com.android.tools.build:gradle：1.0.0'部分意味着:

在仓库中找到com.android.tools.build的手工品组
找到组内的gradle手工品
确保我们的是手工品的1.0.0版
当你首次运行你的构建的时候，有着如上所示的构建脚本闭包,Gradle会提醒你没有这个依赖。它会访问MavenCentral并且找到com.android.tools.build组以及组内的gradle的手工品,然后下载1.0.0版本。

有时候，版本的最后部分被一个加号符号所替代。这就告诉Gradle下载最新版本，从而自动升级到最新的补丁等级。（例如，某个时间点的1.0.3）

####应用插件
com.android.tools.build:gradle：1.0.0依赖包含了一个GradleAndroid插件的依赖，教导Gradle Android特有的结构，例如会在这章之后涵盖到的android闭包里发现的内容。但是，仅仅下载依赖并没有应用任何东西。

apply plugin：’com.android.application'声明真正为Grale构建添加了Android插件。尤其，指示了我们构建了一个Android应用。

####依赖
构建脚本中依赖闭包详述了构建脚本的依赖。build.gradle文件的根中的依赖闭包，对比而言，详述了应用的依赖。

但是，“依赖”的定义大都是相同的:它是一些我们需要添加到我们项目源代码中的类库或相似的手工品。

我们将在这本书之后的内容了解这些类库的概念。

####android
android闭包包含了所有Android特有的配置信息。这个闭包是Android插件提供的。

但是在我们了解这个闭包里面有什么之前，我们应该改变下主题并讨论下manifest文件，因为android闭包中的内容和manifest中的内容有关。

