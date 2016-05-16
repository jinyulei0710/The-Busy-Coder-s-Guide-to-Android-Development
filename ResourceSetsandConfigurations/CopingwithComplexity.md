####应对各种错综复杂的状况

当你需要使用多个用于你的资源的毫不相干的条件的时候，事情就开始变得复杂了。

例如，假定你有着依赖于地域的drawable资源，例如一个停止标志。你可能想要让drawable资源集与语言
进行绑定，所以在不同的地域中使用不同的图片。但是，你可能也希望这些图片会因密度的不同而不同，
较高密度的设备上使用较高分辨率的图片，所以图片出来的话大致会是相同实际大小。

为了这么做，你要产生带有多个资源集限定符的多个目录，例如：

* res/drawable-ldpi/
* res/drawable-mdpi/
* res/drawable-hdpi/
* res/drawable-xhdpi/
* res/drawable-en-rUK-ldpi/
* res/drawable-en-rUK-mdpi/
* res/drawable-en-rUK-hdpi/
* res/drawable-en-rUK-xhdpi/
* 等

一旦你进入这些类型的情形中，一些规则就开始生效了，例如:

1.配置选项有着特定的优先顺序，并且在目录中它们必须以那个顺序出现。Android文档
中概述了这些操作能够出现的特定顺序。就这个样例的目的来说，屏幕尺寸会比
屏幕方向重要些，比屏幕密度重要些，比是否有键盘重要些。

2.每个目录上每个配置选项种类的值只能有一个

3.选项是大小写敏感的

例如，基于屏幕尺寸和方向你可能想要有不同的布局。因为在资源系统中屏幕
尺寸是比方向重要的，屏幕尺寸在目录名中会出现在方向之前，例如：

* res/layout-sw600dp-land/
* res/layout-sw600dp/
* res/layout-land/
* res/layout/
