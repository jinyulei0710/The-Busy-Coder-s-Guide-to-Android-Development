###从哪获取你的碎片

大部分开始这会使用始于API等级11的碎片实现。你会使用android.app.Fragment并让它被你的常规activity所持有。

但是，在Android支持包中有一个向后兼容的碎片系统。这个系统能向后兼容到API等级4，所以如果你的最小sdk版本比11小并且你想要使用碎片的话，向后兼容就是你所要考虑的了。你会需要添加support-v4 JAR包到你的项目中。你也需要使用android.support.v4.Fragment而不是使用android.app.Fragment。同样的，你会需要用继承自android.support.v4.app.Activity的Activity持有这些碎片。

这本书主要关注于使用原生的API等级11的碎片实现，偶尔会有例子由于某种原因向后兼容是必要的。

