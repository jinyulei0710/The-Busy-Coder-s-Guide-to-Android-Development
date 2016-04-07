###创建一个新结构的项目
在写这段的时候，主要有两种方式把项目弄成新的项目结构：使用Android Studio或者手动创建。

正如在这本书最早的章节提到的，Android Studio的原生构建系统是Gradle和Gradle Android 插件。当你通过集成开发环境创建一个新的项目的时候，它会自动设置成新的项目结构。。

不幸的是，写到这段的时候，没有从Eclipse中把项目组成成新结构的方法，只是因为Eclipse不支持新的结构。相似的，也没有类似于android create project这样的命令行工具会创建一个Gradle项目。

因此，短期来说，如果你不使用AndroidStudio,并且你想要新结构的项目，你自己需要手动创建目录树和build.gradle文件。这可能要从零开始，或者是从现有源代码中复制项目结构。Martin Liersch(又名，"Goddchen")发布了一个有多种样例项目的Github库。你可以将它和呈现在本章剩余部分的样例作为灵感源泉。