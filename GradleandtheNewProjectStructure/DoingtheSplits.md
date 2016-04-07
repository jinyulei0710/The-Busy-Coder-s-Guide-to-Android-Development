###做分离
给Android的Gradle 0.13 引进了splits，作为产品风格在两种情形下的轻量级灌装替换：

* 不同的APK文件有不同的NDK二进制文件
* 不同的APK文件针对特定的屏幕密度，这对那些有很多图片并突破发行渠道的上限。(例如，PlayStore上是100MB的限制，较早之前是50MB的限制)

作为一个开发者你所要是在有限配置下，实现特定的分离。注意，你没有分开的Gradle配置（例如，applicationId）以及源集给分离（splits）。这就允许分离在构建的时候处理地更快，因为构建工具 可以做一些简化假定并且避免了重新编译。

####列出分离的范围
分离，默认情况下，会每个可能的类型输出一个APK。例如，在密度上分类会给你为ldpi,mdpi,hdpi,xhdpi,xxhdpi和xxxhdpi生成一个APK。另外，密度这种情况下，你也获得了一个包含默认支持所有密度的“universal”APK。

这样很好...但是如果你不需要为所有这些密度生成APK？例如，你没有装载tvdpi资源，就没有必要为它设置一个hdpi APK。

有两种基本控制构建范围的模式:

使用exclude声明，在初始默认的上移除一些选项
使用reset()方法抹除默认的，然后使用include声明列出你想要的。
换句话说，exclude实现了一个黑名单，reset()/include的组合实现了白名单。所有一切都一样，白名单可能是更好的选择，那么你可以指出什么在你的应用中写了。

####要求NDK分离
在你的android闭包中，你可以添加一个包含一个abi闭包的splits闭包，它按CPU架构顺序设置APK。

	splits{
 	 abi{
   	 enable true
    	reset()
    	include 'armeabi-v7a','x86'
    	universalApk true
  	}
	}
	
这里，我们: * 打开了split(enable true) * 移除了默认包含的abi * 列出了你要包含的ABI(包含‘armeabi-v7a','x86') * 要求也生成一个“全体的APK”，包含所有的ABI(universalApk true)

后者对不允许为不同CPU架构上传多个APK的发行渠道是很有用的。这样，你至少能把你的应用发布到那，即使占用了更多的磁盘空间。默认情况下，由于CPU架构，你不会得到一个“全体APK”。

####要求密度分离
相同的基本模式在密度上也适用:	

 	splits{
	density{
  	 enable true
  	 reset()
   	include 'hdpi', 'xhdpi', 'xxhdpi'
	}
	
再一次，我们打开分离，重置默认，然后选择我们要的密度。

注意，虽然，“全体APK”总是会为所有的密度生成。我们不需要有universalApk true，并且当前universalApk false不是一个可选项。




	
	