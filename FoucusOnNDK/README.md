##关注NDK
当Android首发的时候，许多开发者想要在它上面运行C/C++代码。除了发布一个二进制可执行文件并
通过一个派生进程运行它之外，几乎没有对于C/C++代码的支持。这个虽然能用，但是却有点笨重，并且
基于进程的接口你的C/C++代码如何与基于Java的UI进行利落的交互。最重要的，二进制可执行文件的使用
的支持并不是很好。

在2009年6月，Android核心小组发布了`Native Development Kit(NDK)`。这允许开发者以一种被
支持的方式为Android应用编写C/C++，通过`Java Native Interface(JNI)`把类库链接到基于
Java的宿主应用上。这给Android开发提供了大量的机会，并且书的这部分内容会分析你该如何利用好NDK
去利用这些机会。

这章会分析如何设置NDK并把它应用到你的项目上。它并没有尝试覆盖所有可能的NDK使用-尤其是使用了像
OpenGL和OpenSL许多框架的游戏应用，是超出本书的范围的。

目录：

* [准备工作](/FoucusOnNDK/Prerequisites.md)
* NDK的职责
* NDK安装和项目设置
* 编写你的MakeFile
* 构建你的类库
* 通过JNI使用你的类库
* 构建并部署你的项目
* `Gradle`和NDK
