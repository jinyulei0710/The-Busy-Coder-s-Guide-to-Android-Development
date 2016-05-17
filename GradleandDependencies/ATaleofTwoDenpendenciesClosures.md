###两个依赖闭包的故事

一个build.gradle文件-或者经典Android项目中的build.gradle文件对-会有着两个依赖闭包。

一个会在buildscript闭包中，并且这个依赖集是构建过程本身的依赖。在这里，我们会列出例如Android构建工具之类的依赖，
它们定义了我们能够配置android选项。

依赖闭包是一个构建脚本并且android列出了用于这个项目的依赖。而这个项目是被这个特定的build.gradle所构建的。

换句话说，buildscript依赖是工具依赖，而常规的依赖是第三方代码的编译时资源。
