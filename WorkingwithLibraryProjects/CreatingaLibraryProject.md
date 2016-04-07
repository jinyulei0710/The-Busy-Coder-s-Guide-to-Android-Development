###创建一个类库项目

Android类库项目，在许多的方面，是像一个常规的Android项目的。它有源代码、资源以及manifest。

它就差没有构建APK文件。作为替换，它代表了一篮子的程序资源，并且Android构建工具知道如何融入常规的Android项目。

如何设置一个项目是取决于你的集成开发环境的。

####Android Studio
让你一个项目变成一个Android类库项目仅仅是一个选择正确的Android Gradle插件的问题。

不选：

	apply plugin: 'com.android.application'
而是使用

	apply plugin: 'com.android.library'

就这些-com.android.library插件现在知道它创建的是一个类库，而不是一个应用。

真正的问题是，你在哪创建这个类库。在许多情形中，你会按项目中的模块的方式去做，其中另一个模块是一个应用。这个涵盖以下两部分:

* 类库是独立的，但有一个样例应用展示了它的使用
* 类库是被应用使用的，并且可能是这个项目中的其它应用模块

添加一个新的模块到AndroidStudio中的最简单的方法是通过`new module wizard`，你能通过从主菜单的`File>New Module弹出这个。这弹出了新模块向导的第一页：

为了以模块添加一个类库项目到一个已经存在的项目中，选择模块类型列表中的“Android Library”，然后
点击下一步执行向导的第二步：

这页收集了一些信息，包括：
* "application name",在这个例子中用的是哪个的不清楚
* “module name”，将是这个项目中的目录，类库就在目录中创建
* "package name",会出现在生成类库的manifest中
* 你的最小Sdk版本

这时候，点击下一步会带你到创建新项目是所看到的相同的新建activity流程。如果你想在这个类库中生成一个activity，那就选择activity模板并提供activity模板配置数据。如果你不想要一个activity，在模板表格中选择“Add No Activity”，然后点击完成完成模块的创建。

最终，新模块向导会给在你项目的指定位置给你你设置好一个新的模块，包括修改过settings.gradle把这个子目录作为项目的一个模块。这时候，你就可以开始在项目本身中使用类库了。

####Eclipse

要在Eclipse中创建一个类库项目，开始先新建一个一般的Android项目。然后在项目属性窗口中（例如，选在项目上右击然后选择属性）,在Android部分，选中“Is Library”的checkbox。选中之后，点击应用。





		