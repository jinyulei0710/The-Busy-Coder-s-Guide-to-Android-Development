###步骤#2-设置Java和32位Linux支持

当你写Andorid应用的时候，你通常使用Java源代码编写。Java源代码然后转换成Android能实际运行的东西(APK文件中的Dalvik的字节码)。

你需要获取和安装Sun/Oracle官方的JavaSE软件开发包。你可以从OracleJava站点获取用于Windows，OS X和Linux的JDK。依照由Oracle或Apple提供的安装指导。在写这个的时候，Android支持Java6和Java7，话虽这样说，某些场景是需要Java7的，因此推荐使用Java7。Java8能用，虽然可能你不得不做额外的工作来配置你的集成开发环境让Java8发出兼容Java7的字节码。

Android也支持OpenJDK，特别是在Linux环境上。

Andorid不支持其它的Java编译器，包括Java GUN编译器（GCJ）

如果你的开发系统是Linux，确保你能运行32位Linux库。这个可能没有在你的Linux发行版本中激活。例如，在Ubuntu14.10上，你可能需要运行如下命令来获取Andorid开发工具所需的32位支持：

sudo apt-get install lib32z1 lib32z1 lib32ncureses5 lib32stdc++6

根据你Linux的版本，你可能需要lib32bz2-1.0。

