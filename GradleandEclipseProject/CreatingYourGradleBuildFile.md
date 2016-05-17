###创建你的Gradle构建文件

你需要一个`build.gradle`文件才能使用`Gradle`构建你的项目。

正如在介绍`Gradle`那章中提及到的，`Gradle`是给Android Studio的原生构建系统。
因此，如果你使用这个集成开发工具的话，你应该必然会获得一个`build.gradle`文件。同样的，
如果你从Eclipse迁移到AndroidStudio的话，使用Android Studio导入向导，因为它比你自己弄的好，
并且会帮助把你的代码组织成Android Studio原生使用的基于源集的项目结构。

现如今即使你使用的不是Android Studio，也有两种获取`build.grdle`文件方法：
从Eclipse项目导出或手动创建。理论上来说从Eclipse上导出是最好的。但是要是使用的Eclipse不支持的话，
你可能就要全部手动创建了。毕竟，正如你看到的，你从Eclipse导出过程得到的东西已经过时了。

####从Eclipse导出
如果你已有一个Eclipse项目，获取`build.gradle文件`给项目的最简单的方式是让ADT插件导出一个给你。

####执行导出
为了导出build.gradle文件，要么选择Eclipse主菜单上的File>Export，要么选择包浏览器中的上下文
菜单的“Export”。不管是哪个，应该会带来一个向导样式的对话框，让你选择你要导出什么：

这里，选择“Generate Gradle build files”.如果没有这一选项的话，你可能用的是较老版本的ADT插件并且需要更新。

点击下一步会带出一个包含所有安装的项目列表，从中你会需要勾选你要导出的项目:

注意，由于向导中的一个bug，你的项目可能没有选中。

一旦你选择了项目，下一步的按钮应该就能点击了。点击它会弹出一个确认向导页：

它应该显示你在向导中勾选的项目。这里，有一个强制覆盖已存在文件的勾选框－
在你之前导出过`Gradle`文件并且希望将用新导出的文件取代它们的时候使用。

点击完成按钮会做导出的工作并且带来一个报告页面

在仔细回顾摘要之后（或仅仅只是忽略了），点击完成关闭向导。

####生成了什么
你要费心的是：
* 一个`build.gradle`文件
* 就像之前那章讨论的一样，`Gradle wrapper`，是以gradlew和／或gradlew.bat文件和gradle子目录
的形式存在的。如果你不使用wrapper的话，除了gradle－wrapper.properties文件之外可以随意删除这些文件。
* 一个隐藏的.gradle目录，包含了Gradle构建过程的缓存数据，例如`build.gradle`文件的解析副本，
如果上次运行之后你没有修改build.gradle文件的话就能运行更快。

####需要修补的
为了能让结果产生的项目能很好地使用Android Studio，改变如下：

* Android插件的版本改为1.0或者更高
* 应用插件声明语句从‘android'改成‘com.android.application'
* 在gradle/wrapper／gradle-wrapper.properties文件中的`distributionUrl`改成https:\//service.gradle.org/distributions/gradle-2.2.1-all.zip
