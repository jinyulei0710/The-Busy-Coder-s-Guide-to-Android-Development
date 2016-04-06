###Gradle:大问题
首先让我们通过一系列常见问题解答调查这是关于什么的方式来打好基础。

####什么是Gradle?
Gradle是用来构建软件的软件，也叫做“构建自动化软件”或“构建系统”。你可能在之前在其它环境中使用过其它构建系统，例如make(C/C++),rake(Ruby),Ant(Java),Maven(Java),等等。

这些工具，通过内在能力和你所教给它们的规则－如何决定什么是创建需要的（例如，基于文件改变）以及如何创建它们。一个构建系统并不直接编译，链接，打包应用。而是分成多个编译，链接，打包去做这个工作。

Gradle使用基于Groovy的领域特定语言完成这些任务。

####什么是Groovy
有很多编程语言是设计成运行在Java虚拟机上的。它们中的一些，像JRuby和Jython,是实现了其它常见的编程语言（分别实现了Ruby和Python）。其它语言的就比较独特，而Groovy就是其一。

Groovy脚本有点像Java和ruby的糅合体。如同Java一样，Groovy支持：

使用class关键字定义类
使用extends创建子类
使用import从外部jar包导入类
使用括号(｛和｝)定义方法
通过new操作符创建对象
如同Ruby一样，虽然：

声明可以作为类的一部分，或者仅仅以命令式风格，就像脚本语言
参数和局部变量不是强类型的
值可以自动拼接成字符串，虽然使用的是稍微有点不同的表达式（Groovy使用“Hello,$name”而Ruby使用“Hello,#{name}”）
Groovy是一门解释性语言，像Ruby而不像Java。Groovy脚本是通过执行groovy命令运行的，传递给它要运行的脚本。Groovy运行时，尽管是一个Jar包，为了操作则需要JVM。

一个Groovy的优点是创建了DSL.例如，Gradle就是一个用来进行软件构建的Groovy DSL。Gradle特有功能似乎有一级语言构造，通常和功能本质难以区分开来。还有,Groovy DSL是大量陈述式的，像XML文件。

从某种程度上说，我们的到了两者的最好的地方：XML形式的定义（通常是更少的标点），还有着延伸到Groovy的能力并在有需要的情况下进行自定义脚本。

####什么是Android必须对Gralde做的事情
Google发布了Gradle的Android插件，它给予了Gradle构建Andorid项目的能力。Google也使用Gradle和Android版Gradle作为AndoridStudio的构建系统。

####为什么我们要迁移到Gradle？
最初的时候，当我们要构建一个应用的时候，这些构建步骤是使用Eclipse和Ant完成的。Eclipse是集成开发环境，但Ant是命令行工具。Eclipse不使用Ant进行Andorid项目构建，而是有着它自己的构建系统。并且我们使用这些工具成功构建了百万以上的应用。这些工具仍然有用，虽然Ant支持正在衰退。

所以，为什么要改呢？

似乎有几个成因，包括:

维护两个独立的构建系统（Ant和Eclipse的本地尝试）变得耗时，并且会因AndroidStudio和其它构建系统的到来变得更糟。因此，Google想要基于Gradle，为IDE和命令行标准化一个单独的构建系统

用Ant脚本构建去做Google构建所需的每件事有点力不从心。

Ant没有提供对“外部手工品”（例如，类库）很好的支持以及对这些类库的依赖管理。但是有着把Maven移植到Ant,或使用Maven自己的构建系统的方法，Google从不赞同这种尝试。在这一方面，Gradle提供了比Eclipse或Ant好得多的支持。并且使开发者放心使用来自多种多样作者的类库。

Gradle设计成以类库的形式集成到IDE,远比Ant集成度高。
至于为什么Google选择Gradle而不是Maven...你就要问Google了。

####Gradle和AndroidStudio的关系？
如上所述,AndroidStudio使用新的基于Gradle的构建系统作为它构建Android项目的本地尝试。但是作为AndroidStudio核心的IntelliJ IDE也有着它自己的构建系统（就像Eclipse有一个一样),IDEA更易更换构建系统。

随着时间的推移，这允许Google专注于支持所有情形的单个构建系统（Gradle），而不是不得不去处理独立构建系统的集合。

####Gradle和Eclipse的关系?
写这本书的时候，Eclipse为Android构建脚本使用Gralde的能力，不管是直接还是座位项目源代码的配置数据导入。Eclipse的ADT插件有着导出项目Gradle构建文件的能力。

Eclipse得到Gradle何种程度的支持是不知道的。最好假设不会在短时间内到来。