##RecyclerView(垃圾回收视图)
视觉表现数据项是许多移动应用很重要的一方面。传统的Android的这项功能，是AdapterView控件家族实现的。但是，它们存在一定局限性，特别是关于列表内容动画变化的高级功能。

在2014年，Google通过Android支持包发布了RecyclerView。开发者可以添加recyclerview-v7到它们的项目中并使用Recycler作为大部分AdapterView家族的替代品。RecyclerView是完全新的一个更灵活的容器，有着很多的钩子和代理从而允许插入的行为。

这有着两个主要影响：

1.Recyclerview确实与其对应的Adapterview强大的多.

2.RecyclerView不能拿来就用，甚至于复现基础的ListView/GridView功能也要写相当多的代码。

在这章中，我们会从零开始学习RecyclerView,从基本操作开始。许多其它地方的ListView会在这复现，来看如何使用RecyclerView完成相同的事情。并且我们会探究一些增加的功能，这些功能或许能让RecyclerView在高端Android应用上值得为之付出努力。

目录：
* [准备工作](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/Prerequisites.md)
* [适配器视图以及对它的不满](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/AdapterViewanditsDiscontents.md)
* [进入垃圾回收视图](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/EnterRecyclerView.md)
* [一个普通的列表](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/ATrivialList.md)
* [分隔线选项](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/DividerOptions.md)
* [网格](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/Grids.md)
* [改变项](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/VaryingtheItems.md)
* [可变的行内容](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/MutableRowContents.md)
* [改变内容](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/ChangingtheContents.md)
* [事物的秩序](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/TheOrderofThings.md)
* [其它好东西](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/OtherBitsofGoodness.md)
* [进击的类库](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/TheMarchoftheLibraries.md)
