##Gradle和任务
一个build.gradle文件教Gradle如何执行任务，例如如何编译一个Android项目。在一个能感知Gradle的集成开发环境（AndroidStudio）之外，你使用Gradle本身运行这些任务。如果你的安装了自己的Gradle备份，你可以使用gradle命令；如果你依赖信任的Gradle Wrapper备份，你可以使用你项目根目录下的./gradle脚本。

对于这本书的目的，gradle命令会被展示，但只是作为./gradlew的替代品。在./gradle中，如果你使用的是Gradle Wrapper脚本你会看到gradle。