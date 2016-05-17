##`Gradle`和`Tasks`
`build.gradle`文件教导`Gradle`如何执行多个任务，例如如何编译一个Android项目。
在一个能感知`Gradle`的集成开发环境（AndroidStudio）之外，你使用`Gradle`运行这些任务。
如果你自己安装了`Gradle`拷贝，你会`gradle`命令行；如果你依赖于一个被信任的Gradle Wrapper拷贝，
你会使用你项目根目录下的./gradle脚本。

出于这本书的目的,`gradle`命令行仅仅是以./gradlew的替代品出现，在你使用`Gradle Wrapper`脚本的
地方可以用`gradle`命令行替代./gradlew。
