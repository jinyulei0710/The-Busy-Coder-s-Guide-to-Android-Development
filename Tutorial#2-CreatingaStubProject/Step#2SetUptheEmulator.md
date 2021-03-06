###步骤＃2-设置模拟器
Android工具中包含了一个模拟器，一个假装成Andrid设备的软件。这对开发来说是很有用的－不仅仅因为你可以在连一个设备也没用的情况下学习Android,也因为模拟器能帮助测试那些你没有的设备设置。

你所要做的第一个决定是你是否现在就要烦设置模拟器镜像。如果你已经又了一个Android设置，你可能偏好在它之上测试你的应用，并在之后的时间点来返回来设置模拟器。

你所要做的第二个决定是你是否要设置x86模拟器支持。大都是的Android设置有的是ARMCPU,但是大都数的开发用的机器有的是x86CPU.ARM模拟器在x86机器上是很慢的，因为每个arm指令都要转换成对应的x86指令才能被执行。但是设置x86模拟器支持就稍微有点复杂了，并且你的开发机可能并不支持。如果你想要尝试立刻设置x86模拟器，在这本书的稍后内容中有一些指导，你可以检查回顾以及跟进。

Android模拟器可以模拟一个或多个Android设置。每个你想要的配置都存储在Android虚拟设备（AVD）中。AVD管理器就是你创建这些Android虚拟设备的地方。

注意Android Studio 现在有了它自己的AVD管理器，而这个是跟Android开发者们常用的是独立开来的。你可能在博客和stackOverflow的答案区等地方，看到过较老的AVD管理器截图。AVD管理器作用还是一样的，就是看起来有点不一样。

为了打开Android Studio中的AVD管理器，从主菜单选择Tools>Android>AVD Manager。

在Android Studio 1.4开始的时候，并没有像之前的Andorid Studio版本一样没有提前为你准备好的模拟器镜像。取而代之的是，类似欢迎界面的一屏：

或者你可能看到一个可用虚拟设备的列表，可能包含了已经为了做好设置的一个：

不管你从哪开始，要定义一个新的Android虚拟设备，点击“Create Virtual Device”按钮，然后就会看到一个“Virtual Device Configuration”向导。

向导的第一页允许你选择一个设置简介来做为你的Android虚拟设备的起点。如果没有满足你的要求的，"New Hardware Profile"按钮允许你定义新的简介。

因为模拟器的速度是跟它们虚拟设备屏幕的分辨率紧紧相关的，你通常要选一个低端设备但也不能低得离谱。例如，一个800x480的手机对很多人来说是相当低的分辨率。但是，有很多设置是这样的分辨率（或者更低），这样作为启动模拟器就很合理。

如果你想要基于已存在的设备简介之上创建新的－修改一些参数而其它不变的话。在选择初始配置之后，点击“Clone Device”按钮，

但是，通常来说，开始的时候使用已存在的配置就很好了，点击“Next”允许你选择要使用的模拟器镜像:

这些需要被下载的会有一个蓝色下载拦截，点击链接就回下载镜像。

对于这本书的教程，你需要API等级为18，19，21，22或23的镜像。镜像要是armabi－v7 CPU架构的（除非你配置了x86模拟器支持）。你不需要“Google APIs”的那个镜像，它们是针对内置Google play服务和像Google Maps等相关应用的。它们之后的“Download”会触发下载创建特定API等级和CPU结构组合的所需文件。

点击“Next”允许你完成你的AVD的配置：

推荐使用默认AVD名，虽然你也可以替换成自己的值。其它的默认值就可以啦。

点击“Finish”会让你回到AVD管理器主页，向你展示你的AVD。然后你就可以关闭AVD管理器窗口了。