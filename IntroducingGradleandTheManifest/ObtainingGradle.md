###获取Gradle
跟许多构建系统一样，要使用它，你需要构建系统的引擎。

如果你只会在AndroidStudio中使用Gradle，集成开发环境会为你处理好Gradle。但是如果你计划在AndroidStudio之外使用Gralde的话。（例如：命令行构建）,你要考虑你的Gradle来自哪了。这对 于没有集成开发环境构建应用是很重要的，例如使用持续集成服务，像Jenkins这样的。

####直接安装
大都数开发者要想在AndroidStudio之外使用Gradle的最终都要直接安装Gradle。

Gradle下载页包含了Gradle的zip压缩包：二进制文件，源代码，或者两者

你可以解压压缩包到你开发机上你要放的位置。

####Linux包
你可能通过linux环境中的包管理器获取Gradle。例如，就有一个Gradle的UbuntuPPA。

####gradle Wrapper
如果你是从其它人发布的项目开始的，你可能在项目根目录发现gradlew和gradlew.bat文件，以及一个gradle目录。这代表了"Gradle Wrapper"。

GradleWrapper由三部分组成: * 批处理文件(gradlew.bat)和shell脚本(gradlew) * 批处理文件和shell脚本所使用的jar包（在gradle/wrapper／目录里） * gradle-wrapper.properties文件（也在gradle/wrapper/目录里）

AndroidStudio使用gradle-wrapper.properties文件，从文件中的distributionUrl属性来决定去哪下载用于你项目的Gradle：


	distributionBase=GRADLE_USER_HOME
	distributionPath=wrapper/dists
	zipstoreBase=GRADLE_USER_HOME
	zipStorePath=wrapper/dists
	distributionUrl=https\://services.gradle.org/distributions/  gradle-2.2.1-1ll.zip

当你创建或导入一个项目的时候，或者你改变配置文件所引用的Gradle版本的时候，AndroidStudio会下载distributionUrl所指向的Gradle并安装它到你项目的.gradle／目录下。这个Gradle的版本就是AndroidStudio使用的。

####规则＃1:只使用你相信的发布URL
如果你从第三方导入一个Android项目，例如这本书的例子－并且它们包含了gradle/wrapper/gradle-wrapper.properties文件，检查下发布url指向哪里。如果它是从service.gradle.org或者来自一个企业内部服务器，它可能是可信的。如果它指向位于其它地方的url，你要考虑性爱你是否真的要使用这个Gradle版本，因为它可能已经被篡改了。

这的批处理文件，shell脚本和jar包是为了支持命令行构建的。如果你使用gradlew，它会使用一个本地安装在项目.gradle目录的的Gradle拷贝。如果没有Gradle拷贝的话，像AndroidStudio一样gradlew会从发布URL下载Gradle。注意AndroidStudio并没有使用gradlew这套规则－它的逻辑是内置进AndroidStudio的。

####规则#2：只使用你真正相信的gradlew
检查一个.properties文件的检查URL是否合法是相对简单的。弄清楚批处理文件和shell脚本就有点不方便了。反编译jar包并弄清楚它就相对困难了。还有，如果你使用了来自某人的gradlew，在Gradle拷贝安装之后，脚本和Jar包是运行在你的开发机器上的。如果代码被篡改了，恶意软件有着你的开发机器和它所能触及的东西完整权限，例如你组织内的服务器。

注意你完全不需要使用GradleWrapper。如果你想要担心这个，在你的开发机器上安装一个Gradle版本并移除GradleWrapper文件。你可以使用gradle命令去构建你的应用（如果Gradle的bin/目录在你的PATH中），并且AndroidStudio会使用你的Gradle安装（如果你教导它去哪找，例如通过GRADLE_HOME环境变量）。