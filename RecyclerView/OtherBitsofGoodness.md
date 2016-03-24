###其他的好东西
引用著名美国专题广告牌的话：”等一下，还有呢!"

除了LinearLayoutManager和GridLayoutManager之外，还有StaggeredGridLayoutManager.使用竖直滚动的GridLayoutManager的话，
行都是一样高的，但是宽度是多变的。使用竖直滚动的StaggeredGridLayoutManager，列都是一样高的，但是高度可能不一样。

三个标准的layout manager同样也支持横向操作，只要改下构造器的一个布尔值就可以。在这些情形中，内容是横向滚动的而不是竖直的。这就不需要第三方
类库像ListView那样的实现了。

并且，当然，你可以实现你自己的Recycler.LayoutManager，而不使用任何内置的。
