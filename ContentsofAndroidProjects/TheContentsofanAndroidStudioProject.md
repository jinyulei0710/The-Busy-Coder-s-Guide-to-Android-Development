###AndroidStudio项目的内容
一个AndroidStudio项目是明显比一个Eclipse项目要复杂的。这个复杂性是为了当你成为一个Android开发专家的时候给你更多力量。

尽管就短期而言，它有点让人困惑。

####根目录
在你项目根目录下，最重要的项是app／目录，它是你应用代码所在的地方。我们会在下一节看看它。

除了app/目录，在根目录下其它值得注意的文件包括:

* build.gradle,如上所述是你项目构建指令的一部分。
* 多种其它的Gradle相关文件(settings.gradle,gradle.properties等等)
* local.properties,指示Android SDK工具的位置
* .imp文件，在里面有着一些附加关于你项目的原数据
####App目录
app目录以及它的内容，是你作为一个开发者所花费大部分时间的地方。你很少需要操作根目录中的文件。

在app目录中最重要的是src的目录，它是你项目源集的根，它会在下一节中描述。

除了src目录，在app／中有一些其它需要注意的项

* 一个build/目录，它会包含构建你应用的输出，包含你的APK文件
* 一个build.gradle文件，是你大都数你的项目特有的Gradle配置会在的地方，来教导AndroidStudio如何构建你的应用。
* 一个app.imp文件，包含了更多的AndroidStudio原数据
####源集
源集是你项目组织“源”的地方。这里，“源”不仅仅是编程语言代码（例如,java）,还有其它构建的输入类型，例如你的资源。

源集中你将会花费大部分你的时间的是main／。你会有一个叫作androidTest的存根源集，它是用来创建unit测试的，会在之后的内容里涵盖到。

在源集中，你可以有:

* Java代码，在一个java/目录中
* 资源,在一个res/目录中
* 资产,在一个assets目录中，代表其它你想要打包进应用安装到设备的静态文件。
* 你的AndroidManifest.xml文件
