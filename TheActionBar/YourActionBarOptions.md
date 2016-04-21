###你的动作栏选项

有许多实现动作栏的方式。你可能会使用从API等级11开始的作为Android一部分的那一个。但是，如果你需要的它们的话，有两个向后兼容方案。

####纯原生

就如之前提及的，运行Android 3.0以及更高版本的系统的设备有着作为硬件一部分提供的动作栏支持，并且这个支持是由Android SDK提供的。例如，有一个ActionBar类，并且你可以通过getActionBar()获取一个它的实例。

但是，这只在运行Android 3.0以及更高版本的设备上有效。如果你在一个较旧设备上调用getActionBar(),你会因认证错误运行时异常而崩溃。认证错误是当你编译通过，而你所编译代码指向的东西并不存在的时候，Android告诉你的方式。

如果你的minSdkVersion是11或者更高的版本，你会能够使用原生的action bar，并且这种方式会在这本书中广泛使用。

####向后兼容

如果你的minSdkVersion版本小于11，你主要会有三个选择：

1.使用Android中的"menu"API，它会添加东西到较新设备的动作栏上，但是在较旧设备上会是传统的“选项菜单”。

2.使用appcompact-v7的动作栏向后兼容方案，2013年8月发布在google的支持包中。

3.使用ActionBarSherlock,一个独立的向后兼容方案，在2011年Android3.0发布之后，由Jack Wharton发布。

这章假设你的minSdkVersion是设置成或者更高的版本的并且你会使用原生的动作栏。appcompact-v7和ActionBarSherlock会在单独的章回里面涵盖到。

注意，在2014年10月份的时候，appcompact-v7类库不仅支持动作栏的向后兼容，同时也尝试向后兼容Google的原质设计样式。通常来说，原质设计在Android 5.0并且使用Theme.Material的使用才会有。
appcompact-v7这章会涵盖这个类库在动作栏和你应用UI其它方面。

