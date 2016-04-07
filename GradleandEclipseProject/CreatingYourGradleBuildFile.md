###创建你的Gradle构建文件

你需要一个build.gradle文件好让你的项目能使用Gralde。

正如在Gradle介绍章节中指出的，Gradle是AndoridStudio的本地构建系统。因此，如果你使用这个集成开发工具的话，你应该会自动得到一个build.gradle文件。并且，如果你从Eclipse迁移到AndroidStudio的话，使用AndroidStudio导入向导，因为它比你选择的好并且也会帮助把你的代码组织成AndroidStudio本地使用的基于源集的项目。

尽管你使用的不是AndroidStudio，现如今也有两种方式获取build.grdle文件：从Eclipse项目导出或手动创建。理论上说从Eclipse上导出是最好的。但是要是使用的Eclipse不支持的话，你可能结果全部手动创建它。毕竟，正如你会看到的，你从Eclipse导出过程获得的东西已经过时了。

####从Eclipse导出
如果你有一个已存在的Eclipse项目，最简单获取一个build.gradle文件给项目的方法是让ADT插件导出一个给你。

####执行导出
为了导出build.gradle文件，要么选择Eclipse主菜单上的File>Export，或者选择包浏览器中的上下文菜单的“Export”。不管是哪个，应该会带来一个像向导式的对话框，让你选择你想导出到哪：

这呢，选择“Generrate Gradle build files”.如果没有这一选项的话，你可能用的是较老版本的ADT插件并且需要更新。

点击下一步会带来一个所有安装的项目列表，在这你会需要勾选你要导出的项目:

注意，由于向导中的一个bug，你的项目可能没有选中。

一旦你选择了项目，下一步的按钮应该就能点击了。点击它会弹出一个确认向导页：

它应该显示你在向导中勾选的项目。这呢，有一个枪支覆盖已存在文件的勾选框－在你之前导出过Gradle文件并且希望将它们用新导出的取代的时候使用。

点击完成按钮会做导出的工作并且弹出一个报告页面

在仔细回顾这里摘要之后（或仅仅只是忽略了），点击完成关闭向导。

####生成了什么
你要烦恼的的是：

一个build.gradle文件
一个Gradle包装，形式是一个gradlew和／或一个gradlew.bat文件和一个gradle子目录，就像之前章节讨论的一样。如果你不使用包装的话，你是可以自由删除这些文件的，除了gradle－wrapper.properties文件。
一个隐藏的.gradle目录，包含了Gradle构建过程的缓存数据，例如一个你的build.gradle文件的解析副本，如果上次运行之后你没有修改build.gradle文件的话就能运行更快。
####需要修补的
为了让导出结果项目能很好使用AndroidStudio，改变如下：

＊ Android插件的版本改为1.0或者更高

应用插件声明语句从‘android'改成‘com.android.application'
在gradle/wrapper／gradle-wrapper.properties文件中的distributionUtl改成https:\//service.gradle.org/distributions/gradle-2.2.1-all.zip
