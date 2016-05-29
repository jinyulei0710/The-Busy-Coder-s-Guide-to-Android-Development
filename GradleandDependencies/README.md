##Gradle和依赖
John Donne 写过这么一句话，没有人是一座孤岛。现如今，几乎没有应用是一座孤岛。
很少有应用能完全避免使用第三方代码库的。大都数应用需要一个来自Android支持包的向后
兼容或其它类(例如，ViewPager)，或者会依赖于`Paly Services SDK`,或者会使用很多第三方jar包
和Android类库项目。

好消息是在你构建你的应用的时候，Gradle添加了很多用于引用这些第三方代码库的能力。虽然它稍微增加
了小部分重用(一个jar包)的复杂性，但是它能很大程度上简化“大型的重用”（例如，一些Android类库项目）。

这章会概述你的应用可以拥有的“依赖”种类以及你该如何配置`Gradle`让其支持它们。

注意：这章所描述的项目不是能被Eclipse使用的，因为在写这部分内容的时候Eclipse并不支持Gradle。

目录：
* [准备工作和警告](/GradleandDependencies/PrerequisitesandWarning.md)
* [依赖？](/GradleandDependencies/Denpendencies.md)
* [两个依赖闭包的故事](/GradleandDependencies/ATaleofTwoDenpendenciesClosures.md)
* [依赖于JAR包](/GradleandDependencies/DenpendingUponaJAR.md)
* [依赖于NDK二进制文件](/GradleandDependencies/DependingNDKBinaries.md)
* [依赖于一个Android类库项目](/GradleandDependencies/DependingUponAndroidLibraryProject.md)
* [依赖于子项目](/GradleandDependencies/DependingUponSubProjects.md)
* [依赖于`Artifacts`](/GradleandDependencies/DependingUponArtifacts.md)
* 从Gradle创建JAR包
* 经由`Flavor`的依赖
* 一个传递依赖的特性
* 经由构建类型的依赖
* 分析一些`CWAC`构件
* 依赖和项目结构对话框
