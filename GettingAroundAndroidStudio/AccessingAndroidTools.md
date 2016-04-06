###使用Android工具
不是所有与Android有关的都是直接在AndroidStudio中的。在一些例子中，工具需要被使多个AndroidStudio用户、Eclipse用户以及没提到的用户所共享。在某些情况下，虽然长远来说可能会把这些工具的功能直接整合进AndroidStudio,但是并没有完成日期。

这有一些你可以通过Tools>Android主菜单选项获取的值得注意的Android相关的工具。

####SDK和AVD管理器
正如我们在教程#1中所看到的，SDK管理器时Android下载AndroidSDK部件的工具，包括：

"SDK平台"版本，允许我们在一个特定的API等级上编译
ARM和(有时候)x86模拟器镜像
文档
更新核心构造工具
等等
你可以通过AndroidStudio主菜单上的Tools>Android>SDK Manager启动SDK管理器，或者点击“droid in a box”的工具栏按钮。

AVD管理器时用来创建模拟基于API等级，屏幕尺寸和其它因素的特定Android环境的模拟器的工具／

你可以通过AndoridStudio主菜单上的Tools>Android>AVD管理器启动应用，或通过点击“droid and a screen”工具栏按钮。

####Android设备检测仪

在这本书的别处，你会看到与Dalivik Debug Monitor Server(DDMS)相关的工具参考，例如使用使用它帮助审查你的所运行应用的内存和线程问题。你也会看到像Hierarchy view的工具参考，在你用代码作出很多改变之后，用来在运行时了解你的用户界面。

在Eclipse中，DDMS和Hierarchy View是视角，通过ADT插件的方式添加。

对于不使用Eclipse的人来说，包括AndroidStudio用户,DDMSh和Hierarchy View 是通过Android Studio设备检测仪这个独立工具获取的。AndroidStudio用户可以通过主菜单上的Tools>Android>Android Device Monitor启动检测仪。

首先是一个启动界面:

然后就是检测仪本身:

如果你在网上阅读了关于从DDMS或Hierarchy的事情，例如在博客或Stackoverflow的答案区，大都数这些功能都应该能通过Android设备检测仪得到。