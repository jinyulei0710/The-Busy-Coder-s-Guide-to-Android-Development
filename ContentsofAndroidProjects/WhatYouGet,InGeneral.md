###一般来说，你得到的东西
项目的文件具体细节取决你选择的集成开发环境。但是，不管你是使用AndroidStudio还是
Eclipse,是有许多共同元素的。

一般来说，你得到的东西
项目的文件具体细节取决你选择的集成开发环境。但是，不管你是使用AndroidStudio还是Eclipse,是有许多共同元素的。

####Manifest
AndroidManifest.xml 是一个描述应用创建以及什么组件被应用供应。你可以把它当作是应用的内容列表，跟书的目录列出出现在书中的多个部分，章节以及附录差不多。

我们会在下一章中更为密切地检查manifest。

####Java
当你创建项目的时候，你为应用主活动提供了完全合格的类名（例如.com.commonsware.android.SomeDemo）。然后你会发现你的项目的Java源代码树就报名目录已经摆放好了，外加一个存根活动子类表示你的主活动。（例如，src/com/commonsware/android/SomeDemoActivity.java）。为了实现你的应用，你是欢迎对这个文件就行修改并添加需要的Java类的。并且我们会在这本书的进程中无数次地对其进行描述。

其它地方－在你通常不会使用的目录中－Android构建工具也会在你每次构建你的应用的时候代码生成一些源代码。代码生成类中其一就有R.java，它对从我们的java代码控制我们的用户界面是很重要的。并且我们将会在真正开始构建应用的时候看到很多对这个R类的引用。

####资源
你会发现你的项目有一个res／目录树。这其中包含了“资源”－随应用仪器打包的静态文件，以原始的或者预处理的形式。一些你会发现或在res下创建的子目录包括:

1.res/drawable/ 给图片(PNG,JPEG,等等)

2.res/layout/ 给基于XML的布局说明书

3.res/menu/ 给基于XML的菜单说明书

4.res/raw 给通用的文件(例如一个音频片段，用户信息的CSV文件)

5.res/values 给字符串，尺寸等等

6.res/xml 其它你想要装的通用XML文件

一些子目录可能有后缀，像res/drawable-hdpi/。这指示这个资源目录只应该在特定环境下使用－在此例中，图片资源应该只在高密度的屏幕上使用。

在这本书稍后的内容，我们会涵盖所有这些内容，并更加详细。

####构建指令
集成开发环境需要知道如何获取这些东西并得到一个Andorid APK文件。一些基于集成开发环境是如何写的以及被知晓了。但是一些细节是你可能需要时不时地去配置的，所以这些细节存储在你会编辑的文件中，从你的集成开发环境通过一种或另一种方式。

在AndroidStudio中，大都数这些知识是保存在一个或多个叫做build.gradle的文件中的。这些是Gradle的构建引擎，是AndroidStudio用了构建APK和其它Android输出用的。

在Eclipse中，这些知识分散在多个文件中，一些是你手动编辑的（例如.project.properties）。一些是只能通过Eclipse修改的（例如.classpath）。


