###创建一个类库项目

在很多方面，Android类库项目是像一个常规的Android项目的。它有源代码、资源以及manifest。

它就差没有构建APK文件。相反，它代表了一篮子Android构建工具知道如何融入常规的Android项目的资源。

你如何设置一个Android类库项目是取决于你的集成开发环境的。

####Android Studio
使一个项目变成一个Android类库项目仅仅是一个为`Gradle`选择正确的Android插件的问题。

使用

	apply plugin: 'com.android.library'

而不是：

	apply plugin: 'com.android.application'

没错-com.android.library插件现在知道它创建的是一个类库，而不是一个应用。

真正的问题是，你在哪创建这个类库。在许多情形中，你会按项目中的模块的方式去做，其中另一个模块是一个应用。
这涵盖以下两部分:

* 类库是独立的，但有一个展示它用法的样例应用
* 类库是被应用使用的，并且可能是被这个项目中的其它应用模块使用

通过`new module wizard`新建模块是添加模块到AndroidStudio中的最简单的方法。
你能通过从主菜单的`File>New Module`带出这个。这带出了新模块向导的第一页：

为了添加一个作为模块的类库项目到一个已经存在的项目中，从模块类型列表选择“Android Library”，
然后点击下一步执行向导的第二步：

这页收集了一些信息，包括：
* "application name",在这个例子中用的是哪个的不清楚
* “module name”，这将是这个项目类库被创建的目录
* "package name",这将生成类库的manifest中
* 你的最小Sdk版本

这时候，点击下一步会带你到与创建新项目是所看到的相同的新建activity流程。如果你想在这个类库中生成一个activity，
那就选择activity模板并提供activity模板配置数据。如果你不想要一个activity，
在模板表格中选择“Add No Activity”，然后点击完成完成模块的创建。

最终，新模块向导会给在你项目的指定位置给你设置好一个新的模块，包括修改`settings.gradle`把这个子目录
作为项目的一个模块。这时候，你就可以开始在项目本身中使用类库了。

####Eclipse 略
