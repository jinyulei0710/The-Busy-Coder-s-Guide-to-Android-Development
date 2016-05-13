###来自Android项目视图的一句话

在这本书的早先内容中，当介绍AndroidStudio的时候，我们看到了Android项目视图。

Android被这么创建的众多原因之一是为了帮助你的管理资源，特别是跨越多个资源的时候。

例如，这里是一张相同Android项目的截屏，但是这次values资源是在树中展现的：

这个树使它看起来仅仅有一个res/values/dimens.xml...但是这个文件不知怎么的有孩子。
其中一个孩子仅仅是光秃秃的一个dimen.xml文件名，而另一个有附加的“w820dp”。

这反映了存在两个版本的dimens.xml的事实：一个在res/values/,一个在res/values-w820dp/:

在Android项目视图中，资源是资源进行组织的，而不是以资源集。例如，
这对于发现所有你要调整资源的一个版本是很有用的。
