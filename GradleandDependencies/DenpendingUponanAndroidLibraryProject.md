###依赖于一个Android类库项目
Android类库项目已经成为了项目之间共享代码的流行方式，因为它们除了Java代码之外还有资源。
对于项目之间的代码重用，一些开发者会完全在内部使用Android类库项目。一些开发者会依赖于
第三方类库项目，例如控件类库或者Google的action bar模式的appcompat-v7向后兼容。一些
开发者会发布它们自己的类库被第三方使用。

其核心是，Android类库项目当使用Gradle构建的时候对比使用Ant或者Eclipse构建的时候，并没有
改变很多。但是，你告诉Android这个项目是一个类库项目，以及你使用类库项目的方式，已经改变了很多。

####创建一个类库项目
就如在Android类库项目那章多提及的，常规Android项目和Android类库项目的主要区别是，就
Gradle配置而言，是你所使用的插件。常规的应用项目会使用com.android.application插件，
而Android类库项目会使用com.android.library插件：

        buildscript{
          repositories{
            mavenCentral()
          }
          denpendencies{
            classpath 'com.android.tools.build:gradle:2.1.0'
          }
          apply plugin:'com.android.library'

          android{
            compileSdkVersion 19
            buildToolsVersion "21.1.2"
          }
        }

源集会表现的和在常规应用中所做的一样，否则你的应用和常规的应用就没区别了。但是
，一如既往的，Android类库项目不是用来创建一个APK文件的，而是作为一个类库的。

####依赖于类库项目

对于依赖一个类库项目你主要有两种选择：

1. 如果类库项目是你自己的，特别是当它只供给单个应用的时候，你可以把类库项目设置成
一个模块或者包含应用项目的子项目，就如我们会在下一节所看到的。

2. 你可以以一个AAR artifact的形式发布类库项目到一些仓库，例如一个本地仓库，然后
以依赖的形式引用AAR。artifact以及仓库的内容会在本章稍后出现。

这些不是互斥的。例如由本书作者发布的CWAC类库，就同时使用了这两种技术。这些项目以
Android类库项目外加一个(或者多个)样例应用展示类库的使用。在debug构建中，
样例应用会以子项目的形式依赖类库。而在release构建中，样例应用依赖的是类库的所发布的AAR。
我们会在之后这章中了解到如何根据构建类型的不同而有不同的依赖。
