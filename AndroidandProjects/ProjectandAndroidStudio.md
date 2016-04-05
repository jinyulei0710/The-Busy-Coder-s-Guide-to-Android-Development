###项目和Android Studio

你可能已经选择使用Android Studio作为你的集成开发工具。

使用Android Studio开发一个项目，你可以从简易编程工具创建一个新的项目，你可以从已存在的一个Android Studio复制到新的，或者你可以导入一个已存在的Android项目到AndroidStudio中。接下来的章节会回顾这些的必要步骤。

####创建一个新的项目

你可以从这两个地方中的一个创建项目

如果当你打开Android Studio的时候，你处于首次碰到的初始对话框，选择“Start a new Android Studio project”菜单按钮。
如果你处于Andorid Studio集成开发环境中，从主菜单选择File>Create Project
这就带来了新项目向导：

向导首页是你可以详细指明以下内容：

* 应用名
* 包名
* 项目放置的文件夹
默认情况下，包名由两部分组成:

1.你在“CompanyDomain”区域详细指明的域名

2.包名，被转成的全部小写无空格和其他标点

如果这不是你想要的，点击建议包名右侧的小小的“Edit”链接，它现在允许你直接编辑包名:

点击“Next”让你来到让你指示创建项目的类型向导页，依据预期的设备类型（手机、平板、电视等等）以及最小所需SDK等级作出选择。

开发者启动Android仅仅需要勾选“Phone and Tablet”作为设备类型。默认的“Minimum SDK”值通常也是个好的选择，并且正如我们稍后再这部书中所看到的，它在你的项目中随时可以更改，

点击“Next”下一步让你来到向导第三页，在这里你可以选择让Android Studio创建初始化活动是基于什么模板：

这些模板中没有一个是特别好的，因为它们添加了大量的你可能完全将其替换掉的示例材料。“Empty Activity”是首次接触Android开发者的最优选项，仅仅因为它加了最少的东西。

如果你选择了除“Add No Activity”之外的选项，点击“Next”将带你来到一个给你提供关于活动如何创建的附加细节：

出现在这里的选项很大程度上取决于你前一页的选择。常见的选项包括“Activty Name”（活动的Java类名）以及“Layout Name”（包含你的活动用户界面的XML文件的基本名）。

点击“Finish”会生成你的项目文件。

####复制一个项目

Android Studio项目只不过是文件目录，没有保存在其它地方的特别元数据（正如Eclipse中）。因此，赋值一个项目，仅仅需要复制它的目录。

####导入一个项目

你可以从以下两个地方中的一个导入项目

* 如果当你打开Android Studio的时候，你处于首次碰到的初始对话框，选择“Import Project”菜单按钮。
* 如果你处于Andorid Studio集成开发环境中，从主菜单选择File>New...>Import Project...
然后，选择项目要导入的目录。

现在发生的取决于项目的特性。如果对Android Studio的使用进行了设置，至少是Android的Gradle，Android Studio详细描述的文件会在项目目录。

但是，如果项目没有对Andorid Studio会Andorid的Gradle进行设置，但是有Eclipse项目文件（或至少有project.properties文件），你会被导向Eclipse导入向导。

向导的第一页是你详细说明Android Studio应该在哪复制这个项目的地方，因此它没有修改过初始目录的任何东西：

点击“Next”会带你来到一个你能对一些作用于导入项目代码的自动修复进行配置。这里的细节已经越过了我们这本书目前覆盖的范围。通常来说，默认的就很好。

点击“完成”会完成项目转换。Android Studio会打开一个import-summary.txt文件列出转换如何完成的一些详细内容。这时候，复制修改的项目已经能使用了。
