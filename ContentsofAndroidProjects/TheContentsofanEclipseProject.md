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

6.隐藏的Eclipse项目文件(例如.classpath)
