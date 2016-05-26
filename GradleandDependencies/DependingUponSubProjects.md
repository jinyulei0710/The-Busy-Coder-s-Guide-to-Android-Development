###依赖于子项目

在经典的Eclipse项目中，我们并没有子项目的概念。但是，Gradle和AndroidStudio都支持一种
包含一系列项目的顶层目录结构。

例如，Gradle/HelloMultiProject目录包含了:
* 一个HelloLibraryConsumer项目
* 一个包含一个Hellolibrary项目的libraries/子目录

在此情形中，HelloLibrary是一个Android类库项目，是来自在这章前面看到的
build.gradle文件。HelloLibraryConsumer是一个常规的应用，它使用的
是HelloLibrary Android类库项目中的代码。

如果你的类库项目仅仅是被一个应用所使用的，你可能会选择这种方式组织你的代码。

顶层目录一定要有一个settings.gradle文件，其中列出目录中所能发现的Gradle项目
(以及它的所有子目录)。

      include ':HelloLibraryConsumer',':libraries:HelloLibrary'

开头的:指的是总的根目录，所以:HelloLibraryConsumer指的是根目录下的HelloLibraryConsumer/
目录。其它的：值代表的是层级结构中的等级，所以，:libraries:HelloLibrary/指的是
libraries/HelloLibrary/subdirectory，而不是使用/,/在Gradle中另有用途。

因此，include声明告诉Gradle这个项目总共由两个子项目构成。

为了指示HelloLibraryConsumer想要使用来自HelloLibrary类库的代码，还有一行
要加到dependencies闭包中，来引用子项目：

        buildscript{
          repositories{
            mavenCentral()
          }
          dependencies{
            classpath 'com.android.tools.build:gradle:2.1'
          }

          apply plugin:'com.android.application'

          dependencies{
            compile project(':libraries:HelloLibrary')
          }

          android{
            compileSdkVersion 19
            buildToolsVersion "21.1.2"
          }
        }

鉴于fileTree()是发现libs/目录中的JAR包，project(':libraries:HelloLibrary')
是识别子项目，并且我们告诉Gradle编译子项目到我们的项目中。

从根目录(那个有settings.gradle文件的目录)，你可以运行整个应用的gradle 任务。
这在主要应用的目录中也有效（在此情形中，HelloLibraryConsumer/)。

虽然这个方式对一些应用来说是很好的，当类库可能被多个应用使用的时候这个结构可能不能很好工作。
特别是类库被其它作者所使用的时候。在这个情形中，类库需要以一个artifact的形式发布到
仓库，会在下一节涵盖到。
