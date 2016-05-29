###依赖于多个`Artifact`

虽然JAR包和子项目是用于Android的Gradle必然使用的，指定依赖的主要方式还是
通过引用放在仓库的多个artifact。

####什么是Artifact？

在基于Java的编程的背景下，artifact通常指的是一个伴有提供版本信息，artifact本身的依赖清单以及相关
信息的元数据JAR包(或者其它编译结果，像JAVA EE WAR包)。

####什么是仓库？

仓库是多个artifact的一个集合，存储在相同的位置，指向这个位置从而发现和解析依赖。

仓库可以进一步分成本地的和远程的。本地仓库是存在于你自己开发机器上的；远程仓库是存在于某个服务器上的。
服务器可以是相对本地的(例如，支持一个公司开发小组的仓库)，或者它也可以存放在其它地方。可能最知名的
其它地方是Maven Central，一个被许多开源项目用于发布它们的多个artifact的仓库。

####Artifacts和仓库的类型
存在两种类型的仓库以及相关的artifact结构，由Gradle支持的：Maven和Ivy。
每个对于元数据和文件存放的结构都有着各自的格式。

#####Maven

[Apache Maven](http://maven.apache.org/)是一个成熟的构建系统。这个构建系统的一部分是多个artifact和仓库
的系统。虽然Gradle不使用Maven的构建系统，或者说在很大程度上取代了Maven.
Gradle能使用发布在Maven-结构仓库的多个artifact。MavenCentral,就像所期待的，
这样这样的一个仓库，但是非常有可能性设置你自己的仓库，并且一些组织已经这么做了。

因为两者之中Maven更加流行，这本书会关注于Maven结构的仓库和Maven结构的artifact。

#####Ivy
[Apache Ivy](http://ant.apache.org/ivy/)是给予我们Ant构建系统的Apache Ant项目的一个
分支。Ivy只不过是一种声明组件之间依赖的方式，包括处理“传递依赖”。

####一般的Artifact依赖设置

要依赖artifact，你需要教导Gradle两件事情：

1.从哪发现artifact?

2.你需要什么artifact?

前一个来自你build.gradle文件中的一个仓库闭包，指定从哪个仓库搜寻artifact。

后面一个来自于你的依赖闭包中的其它关于compile声明的变量。你可以从你所指定的仓库中
使用artifact而不是使用像fileTree()的东西。

Artifacts由三个部分的数据所标识：

1.一个group

2.一个artifact ID

3.一个版本号

这些由冒号分隔开，所以compile 'com.commonsware.cwac.colormixer:0.5.0'
表明你在寻找这样的artifact：

* ...在com.commonsware.cwac group中
* ...有着colormixer的artifact ID，以及
* ...版本号是 0.5.0

####依赖于Maven Central或JCenter的Artifacts

最常去获取artifact的地方是Maven Central。这大概类似RubyGems仓库之余Ruby开发者，
又或是CPAN之余Perl开发者。Maven是一个大量artifact的仓库，其中只有一部分是与Android相关的，
因为Maven Central也被其它java开发环境所使用(例如Java Web容器)。

Bintray-Jcenter属于它的artifact仓库业务。JCenter是Maven Central的
一个镜像，并且开发者可以直接发布artifact到JCenter。

如果你想要使用它们中的一个仓库，你的在顶层build.grale文件会需要一个仓库闭包，
这个闭包会请求mavenCentral()或者jcenter().

       repositories{
          mavenCentral()
       }

然后你就可以在你的依赖闭包中加上你所能在MavenCentral或JCenter所能找到的
artifact。其中一种发现你能include什么的方法是访问["Gradle,Please"](http://gradleplease.appspot.com/)
这个网站.在这个网站中，你可以输入一个流行类库的名称(例如，otto)，你就能得到对应的依赖闭包：

      denpendencies{
        compile 'com.squareup:otto:1.3.4'
      }


如果你有着多个依赖，把它们一个接一个的列在一个依赖闭包中。所以，例如，
要同时依赖Otto和本地JAR宝，你会有如下的依赖闭包：

     denpendencies{
       compile 'com.squareup:otto:1.3.4'
       compile fileTree(dir:'libs',include:'*.jar')
     }

你可能回想起一个Android Studio项目的顶层build.gradle文件会包含像一些这样的东西：


     allprojects{
       repositories{
         jcenter()
       }
     }

allprojects闭包结构代表着配置应该被应用到项目中所有的模块(子项目)。因此定义在这里的
repositories应该被应用到所有的模块，超出模块中自己的build.gradle文件。

     
