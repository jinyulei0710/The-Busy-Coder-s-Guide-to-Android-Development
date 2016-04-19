###进入垃圾回收视图
RecyclerView就是为弥补这些瑕疵而设计的。

RecyclerView它自己本身除了帮助管理视图回收（例如：竖直列表的行回收）之外做了很少的工作。它几乎所有其它的东西给委托给了其它的类，例如:

* layout manager，负责把视图组织成多样的结构（竖直列表，网格，交错网格，等等）
* item decorator，负责给视图应用效果和光定位，例如在竖直列表之间加上分隔线
* item animator，负责在模型数据改变的时候的动画效果

这是在需要你熟练掌握adapters和view holders（常见AdpterView使用方法的优质标志）的。

因为layout manager是通过抽象类和可更换的具体措施来进行处理的，第三方开发者可以选择共享给开发者使用，就像Google做的那样。
[在本章稍后内容中](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/TheMarchoftheLibraries.md)，我们会来探索一些这种第三方类库。

另一方面，虽然RecyclerView没有像ListView或GridView那么现成可用。在recyclerview-v7库中，并不是所有东西都没了，而是需要你自己动手实现或依赖于第三方类库实现。


