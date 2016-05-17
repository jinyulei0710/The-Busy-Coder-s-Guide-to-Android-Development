###结果

使用Eclipse的话，构建结果会到bin/目录，并生成源代码（例如R.java）到gen/目录。
使用Gradle的话，所有的输出结果都会到build/目录。

具体来说，你的apk文件生成到build/outputs/apk目录中。因构建行为是debug还是release而构建生成
不同的APK版本。

Gradle有一个清理build/目录的clean任务。但是,注意Eclipse不使用build/目录，
所以`Gradle clean`操作不会清理Eclipse使用的预编译类。
