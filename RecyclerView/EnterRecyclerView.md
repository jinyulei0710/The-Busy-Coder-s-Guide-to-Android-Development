###进入垃圾回收视图
RecyclerView就是为纠正这些瑕疵而设计的。

RecyclerView它自己本身除了帮助管理视图回收（例如：竖直列表的行回收）之外做了很少的工作。它委托几乎所有其它的东西给其它的类，例如:

* 一个布局管理器，负责把视图组织成多样的结构（竖直列表，网格，交错网格，等等）
* item装饰，负责给视图应用效果和光定位，例如在竖直列表之间加上分隔线
* item动画，负责在模型数据改变的时候的动画效果

这是在需要你熟练掌握adapters和view holders（常见AdpterView使用方法的优质标志）的。

因为想布局管理器是通过一个抽象类处理的并且把一些具体的实现放回去了，第三方开发者可以选择共享给开发者使用，就像Google做的那样。
[在本章稍后内容中](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/RecyclerView/TheMarchoftheLibraries.md)，我们会来探索一些这种第三方类库。

另一方面，虽然RecyclerView没有像ListView或GridView那样立即可用。在recyclerview-v7库中，并不是所有东西都没了，
你可以选择自己实现也可以依赖第三方类库来完成任务。
