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

####依赖于Google的Artifacts

但是不是所有的artifact都是存储在MavenCentral或JCenter的。

一个存在其他地方的artifacts就是Google的。它们提供了它们自己的仓库，而不是依赖于MavenCentral，
你可以通过SDK Manager下载这个仓库。它们被称为“Android 仓库”以及”Google仓库“，前者是用于
像Android support包的，后者是用于PlayServices DK的。

这些仓库，一旦找到，就会自动添加到你的Gradle环境中。而不像MavenCentral需要你手动添加它们到
依赖闭包中。

如果你已经读过了这本书的核心章回，你会已经了解了support-v4和support-v13artifacts，提供了
像ViewPager和NotificationCompact这样的类。Android仓库和Google仓库之间存在很多其它结合，
其中很多会在这本书其它地方讲到。

通过引用这些artifacts，你再也不同区烦恼复制和添加Android类库项目到你自己的
项目中去了。

####依赖于其它仓库

Gradle支持自定义的Maven或Ivy样式的artifact仓库来获取artifact。例如，你的开发团队可能有个
仓库用于你的项目，由开发者和持续集成服务器共享。

[Gradle文档](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html)涵盖了不同的可能性，
例如对于受保护的仓库如何包含认证凭据，尽管通常来说，你会使用一个简单的URL：

        repositories{
          maven{
              url =“http://repo.commonsware.com”
          }
        }
这个仓库闭包添加了一个位于http://repo.commonsware.com的Maven样式的仓库
       repositories{
          maven{
            url =“https://repo.commonsware.com.s3.amazonaws.com"
         }
        }

这是一个等价的仓库闭包，但是为了安全下载artifact是以https作为方案的。因为这个特定的仓库
恰好存放在Amazons S3，SSL证书需要我们使用完整的Amazon S3域名而不是CNAME简写。

自此，请求编译任何发布在仓库的artifact就变得可行了。

####你自己的仓库

你可能想要拥有仅仅位于你自己开发机器上的本地的仓库。例如，你编写了一个Android类库项目，
以子项目引用的方式是不合适的(例如，这个类库被多个应用使用)，你可以发布你的AAR Artifacts
到你本地的仓库，然后让你其它的应用依赖于所能在这个仓库找到的artifact。

使用你本地的仓库仅仅是一个添加一个mavenLocal到你的依赖闭包中的问题：
       repositories{
         mavenLocal
       }

仓库的准确位置是与平台有关的。例如在Linux上，它就位于~/.m2/,但是它是位于
你本地的机器上。

注意：在某些版本的Gradle上，mavenLocal()卜启晓。变通的方法是：

      repositories{
        maven {url "${System.env.HOME}/.m2/repository"}//mavenLocal()
      }

如果你想的话，你可以有自己的仓库。例如，这本书的作者正慢慢地把它的JAR包和AAR形式的CWAV项目
从之前提及的repo.commonsware.com仓库转移出来。在Gradle的视角看来，临近的文件服务器和一些
远程的Web服务器并没有区别     

####把类库发布成Artifacts

当然，拥有本地(或者远程)仓库只和你把东西放到仓库一样好。并且，现在是当前的GradleAndroid插件
跌跟头的地方。文档提及了AAR格式但没有提供与发布相关的指导，并且插件中的变更以及破坏了许多自2013
年以来的拼凑解决方案。

放弃最简单的办法是以maven插件的形式，不言而喻，是一个从Gradle构建添加发布Maven Artifacts
支持的Gradle插件。

然后，你就可以通过一个upliadArchives作为顶层闭包配置在哪里以及如何Maven插件发布你的AAAR：

        apply plugin: 'maven'

        uploadArchives{
           repositories.mavenDeployer{
             pom.groupId='com.commonsware.cwac'
             pom.artifactId='everything-is-awesome'
             pom.version='0.0.1'

             repository(url:'file://home/somebody/put/a/real/path/in/here/kthxbye')
           }
        }
groupId和artifactId是你的actifact的基本组成。verison是artifact的版本号。url参数说明了应该
往哪上传artifact，这可以是公共的仓库(例如MavenCentral)，一个私有的企业持有的仓库，一个本地的仓库
，等等。

在此刻，gradle uploadArchives 命令会构建你的AAR并把它部署到你所指明的Maven仓库。

####把旧式结构的类库发布成Artifacts
注意没有说一定要从使用新的构建系统偏好的目录结构才能创建AAR。你的AAR可以来自于保留有旧式
目录结构的项目。尽管AAR支持慢慢变得占据主导地位，这是接下来几年的关键。这样你才能支持
你的Android类库也被宰传统的代码形式中使用。

你会在这章之后看到用于ARR创建的类库项目使用旧式结构的例子。

####关于更新

你所获得的artifact版本是由你声明依赖的版本决定的。Gradle本身不存在告诉你关乎有新的artifact更新的
任何东西。Ben Manes 发布了[一个Gradle插件](https://github.com/ben-manes/gradle-versions-plugin)。
这个插件添加了一个依赖更新任务，这个任务生成了你的依赖状态的一份报告。
