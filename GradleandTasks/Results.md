###结果

使用Eclipse的话，构建结果会到bin/目录，并生成源代码（例如R.java）到gen/目录。使用Gradle的话，所有的输出结果都会到build/目录。

具体地，你的apk文件会到build/outputs/apk。因构建行为是debug还是release而构建生成不同的APK版本。

Gradle有一个clean任务清扫build/目录。但是,注意Eclipse不使用build/目录，所以Gradle cleanc操作不会清理Eclipse使用的预编译类。

