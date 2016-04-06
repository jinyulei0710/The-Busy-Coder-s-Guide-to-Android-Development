步骤#1:支持多个屏幕大小

我们的应用会限制它所支持的屏幕大小。平板是电子书读者的理想载体。手机也能用，但是
比手机还小的，就更加困难做出一个能让用户做所需的事情的界面了，而且还要留下一到两行的空间。

我们会在这本书稍后内容了解屏幕大小策略和它们的详细内容。尽管当前，我们会添加一个
<supports-screens>元素来让我们的应用抛弃小屏设备(小于3寸)。

Android Studio 用户可以在项目浏览器中对AndroidManifest.xml进行双击。

添加一个<supports-screens>元素作为根<manifest>元素的一个子元素，如下所示

	<supports-screens
	  android:largeScreens="true"
	  android:normalScreens="true"
	  android:smallScreens="false"
	  android:xlargeScreens="true"/>


	  
