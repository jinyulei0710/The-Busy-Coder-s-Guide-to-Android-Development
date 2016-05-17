##Gradle和新项目结构

前一章展示了如何使用Gradle，以及Android Gradle插件，用命令行,Eclipse，ItelliJ IDEA,Ant
等构建项目。

虽然传统的项目目录结构能用，但是它并不能让发挥Android Gradle插件的全部威力。为了利用新构建系统
的灵活性,你将需要以稍微有点不同的方式组织你的`source`，`resources`，`assets`以及相关的文件。

这章会列出这个“新的项目结构”的概要，并向你展示给Android的Gradle的`build type`和`product flavors`
是如何让你更简单地从单个尽管重组的项目树得到多种形式的输出。这个项目结构是AndroidStudio来说是原生的，
所以Android Studio项目已经做好了支持这些种类的高级功能的准备了。

注意：这章所描述的项目不再能被Eclipse使用了，因为Eclipse不支持Gradle。
