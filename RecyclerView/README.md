##RecyclerView(垃圾回收视图)
视觉表现数据项是许多移动应用很重要的一方面。传统的Android的这项功能，是AdapterView控件家族实现的。但是，它们存在一定局限性，特别是关于列表内容动画变化的高级功能。

在2014年，Google通过Android支持包发布了RecyclerView。开发者可以添加recyclerview-v7到它们的项目中并使用Recycler作为大部分AdapterView家族的替代品。RecyclerView是完全新的一个更灵活的容器，有着很多的钩子和代理从而允许插入的行为。

这有着两个主要影响：

1.Recyclerview确实与其对应的Adapterview强大的多.

2.RecyclerView不能拿来就用，甚至于复现基础的ListView/GridView功能也要写相当多的代码。

在这章中，我们会从零开始学习RecyclerView,从基本操作开始。许多其它地方的ListView会在这复现，来看如何使用RecyclerView完成相同的事情。并且我们会探究一些增加的功能，这些功能或许能让RecyclerView在高端Android应用上值得为之付出努力。


###准备工作

为了理解这章需要你已经读了核心章节，特别是[AdapterViews和adapters](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/tree/master/AdapterViewsAndAdapters)那章。

一节包含了[自定义XMLdrawables](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/tree/master/CustomDrawables).另一节展示使用的内容是从[MediaStoreProvider](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/tree/master/TheMediaStoreProvider)拉取的。

这章也包含了像[action modes](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/tree/master/ActionModes)和其它的[高级ListView技巧](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/tree/master/AdvancedListViews)。

###AdapterView以及对它的不满

AdapterView，特别是它的子类ListView和GridView,在Android应用开发中承担了重要的工作。并且，对基本场景来说，它们还是相当令人满意的。

但是，也存在问题。

可能最大的策略问题是更新一个AdapterView倾向于要么全部更新要么不更新。如果模型数据改变了-新的行添加了，存在的行移除了或者数据改变,这些可能影响AdapterView的展现-唯一提供很好支持的解决方案是调用notifyDataSetChanged()并让AdapterView重新构建自己。这是缓慢的并且可能对选择状态有影响。并且如果你真的要在复杂的改变，并且在行添加或移除的时候使用动画效果，一半是不可能的。

策略上，AdapterView,AbsListView(ListView和GridView的直接父类)，和ListView对许多门外汉来说就是一大堆跟pasta一样的代码。这些类中堆了太多的责任，以至于对Google来说可维护性是一种挑战，扩展性就有点不切实际了。

目录：
* [准备工作]()
* [适配器视图以及对它的不满]()
* [进入垃圾回收视图]()
* [一个普通的列表]()
* [分隔线选项]()
* [网格]()
* [改变项]()
* [改变行内容]()
* [改变内容]()
* [事物的秩序]()
* [其它好东西]()
* [进击的类库]()
