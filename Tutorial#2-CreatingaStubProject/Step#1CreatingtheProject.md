###步骤#1：创建项目
我们需要为EmPuLite创建Android项目。

首先，访问这本书Github仓库的发布区域并下载跟书版本对应的EmPubLiteStarter-AndroidStudio.zip文件。

然后，解压zip压缩文件到你开发机上集成开发环境之外的某个目录。ZIP压缩文件会解压成一系列文件和子目录。最有可能的是，你会想要把这些房间一个空的已存在的文件。Eclipse用户不应该把压缩包直接解压到Eclipse工作空间，因为这很容易让Eclipse困惑。

在AndroidStudio中，你可以从下面两个地方中的一个导入:

如果你处于你在首次打开AndroidStudio的欢迎对话框中，选择“Import project(Eclipse ADT,Gradle,etc)”菜单项。注意，这回道了首个屏幕－欢迎对话框的快速开始，所以如果你在对SDKManager的这样一些选项进行配置，点击左上角左箭头返回到快速开始页。

如果你处在AndroidStudio集成开发环境中，从主菜单选择File>New>Import Project。

然后，只要选择你解压EmpuliteStater－AndroidStudio.zip的目录。

因为这个项目已经对Android Studio的使用做好设置，你应该被带进主集成开发环境。

你可能会在消息窗口看到一个错误，像“failed to find Build Tools revision...”这样的消息。Andorid Studio和Android SDK随着这本书的版本更新不断发展。每个Android项目包含了你想要使用的编译工具的版本信息。然而，你导入的起始项目使用的是在这本书的更新时候最新的编译工具，这个版本可能和你从AndroidStudio获取的版本不同。如果这个错误消息出现了，它应该会有带着一个像“Install Build Tools...and sync project”的链接。点击那个链接并等待一会,AndroidStudio在为你的项目下载缺失的东西。

Android Studio有两种查看Android项目内容的方式，默认的一个是，导入项目之后呈现的是，名为“Andorid项目视角”:

虽然，你是欢迎用这种视图浏览你的项目的，教程中的Android Studio截图都是以传统视图展现的：

要切换到传统视图，对应教程所展现给你的－点击当前展示为“Android”的下拉列表然后换成“Project”。