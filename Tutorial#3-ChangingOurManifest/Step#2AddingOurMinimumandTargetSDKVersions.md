###步骤#2：添加我们的最小和目标SDK版本

我们也需要教导我们的集成开发环境我们的最小SDK版本(我们会支持的Android版本有多老)以及
我们的目标SDK版本(编写代码时所设想的Android版本)。

经典的做法的话，我们会以在manifest中添加一个<uses-sdk>元素的方式完成。现如今，
Android Studio用户也可以完成相同的事情，但是作为替代的，我们会遵循AndroidStudio
的惯例并添加这些值到build.gradle中。

在项目浏览器中的app/build.gradle上双击，离开根项目

你应该会看到一个像这样的defaultConfig闭包:

	defaultConfig{
		applicationId ="com.commonsware.empulite"
		versionCode 1
		versionName "1.0"
	}

添加minSdkVersion 15以及targetSdkVersion18声明到它里面，那么它看起来像这样：

	defaultConfig{
		applicationId ="com.commonsware.empublite"
		versionCode 1
		versionName "1.0"
		minSdkVersion 15
		targetSdkVersion 18
	}


你会在编辑器的顶部看到一个黄色标语,指示有一个项目同步请求。如果看到了，点击标语中的“Sync Now“链接来同步
你对这个build.gradle的修改到*.iml文件中。如果你没有看到这个标语，从主菜单中选择Tools>Android>Project Sync With Gradle
Files来完成相同的事情。你会想要在每次你改变build.gradle的时候进行同步，来确保Android其余部分注意到了你的改变。


