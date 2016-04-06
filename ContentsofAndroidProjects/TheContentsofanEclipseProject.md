###Eclipse项目的内容
当你在Eclipse项目中创建项目的时候，你会在项目的根目录获得一些项,包括:

1.如上所述的AndroidManifest.xml

2.bin/,包含应用一旦编译之后的东西（注意：这个目录会在你首次构建你的应用时生成）

3.res/，包含了如上所述的你的资源

4.src/，包含了应用的java源代码

除了以上的文件和目录，你可能在Android项目中发现以下的任何一个:

1.assets/,包含了其它你想要打包进安装在设备的应用中的

2.gen/，存放Android的构建工具生成的源代码

3.libs/,存放你应用需要的人和第三方java的JAR包

4.＊properties,包含了你构建的配置数据

5.proguard.cfg或proguard-project.txt，用来集成ProGuard来混淆你的Android代码

6.隐藏Eclipse项目文件(例如.classpath)

你从它那得到了什么
作为你运行你的应用在设备或模拟器上的一部分，集成开发环境会生成一个APK文件。你会发现这个：

* 在你的AndroidStudio项目的build/outputs/apk目录中，如果项目没有单元（例如没有app目录），或者
* 在你的单元的build／outputs/apk目录中，（例如，传统AndroidStudio项目的app/build/outputs/apk）,或者
* 在你的Eclipse的bin目录
 
APK文件是包含你编译Java类的zip压缩包，你的资源的编译版本（resource.arsc）,任一的未编译的资源（例如你放进res/raw/文件夹的),以及AndroidManifest.xml文件。如果你构建了调试版本的应用－默认的。对于一个叫作yourapp的应用来说，你会得到你的app－debug.apk作为你的APK。