###你从它得到了什么
作为你运行你的应用在设备或模拟器上的一部分，集成开发环境会生成一个APK文件。你会发现这个：

在你的AndroidStudio项目的build/outputs/apk目录中，如果项目没有单元（例如没有app目录），或者
在你的单元的build／outputs/apk目录中，（例如，传统AndroidStudio项目的app/build/outputs/apk）,或者
在你的Eclipse的bin目录
APK文件是包含你编译Java类的zip压缩包，你的资源的编译版本（resource.arsc）,任一的未编译的资源（例如你放进res/raw/文件夹的),以及AndroidManifest.xml文件。如果你构建了调试版本的应用－默认的。对于一个叫作yourapp的应用来说，你会得到你的app－debug.apk作为你的APK。