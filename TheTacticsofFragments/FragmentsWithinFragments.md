###碎片中的碎片:仅仅说是有可能

历史上，碎片的一个主要限制是它们不能再包含其它碎片。在大多数情况下，这不会产生一个较大的问题。
但是，有时候你可能要被这个限制绊倒，例如在使用ViewPager的时候，会在稍后一章进行讲解。

Android 4.2-以及Android支持包-添加了嵌套碎片的支持。然而一个activity通过一个FragmentManager进行碎片的使用，这个FragmentManager呢是通过getFragment或getSupportFragmentManager()获得的，碎片能够通过调用getChildFragmentManager()使用嵌套碎片。


但是，Android 3.0-4.1之间的这个版本的碎片并没有getClildFragment()方法。因此，你有了两个选择:

1.使用Android支持包的碎片向后兼容，直到你可以不对Android 4.1或更早的版本进行支持

2.暂时不要使用嵌套碎片。

我们会在ViewPager这章看到getChildFragmentManager()是如何工作的。




