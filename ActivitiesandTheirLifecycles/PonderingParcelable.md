###考虑Parcelable

就像以上提及到的，`Intent extras`不能处理任意的对象的。这是因为，大都时候，`Intent extras`是跨进程传递的。
即使你调用startActivity()来启动你自己的多个activity中的一个，请求从你的进程传递到核心系统进程然后在返回到
你的进程。同时`Intent extras`也传了过去。


因此，`Intent extras`需要是能够转换成一个二进制数组的东西，作为进程间通信的一部分处理传入的Intent对象。你也会看到在其他风格的Android IPC看到这个，例如远程服务。

但是，有两种方式你可以尝试，使你的对象以`Intent extras`的形式工作。

一种方式是在你的类上实现`Serializable`。这是经典的Java结构，设计成允许你的类的实例和你的实例所持有的序列化的对象，序列化到文件中然后之后从文件中读取出来。


另一种方式是在你的类上实现`Parcelable`。这是一种Android结构，这个结构和`Serializable`非常相似。但是，`Serializable`是为对象的持久化存储而设计的，可能几个月或者几年之后都可以从文件中读取回来。同样的，`Serializable`需要处理java代码上实现这些类的改变，并且同样也需要有着帮助转换旧的，保存过的对象到新的对象上的钩子。这就增加了开销。`Parcelable`只关心把对象转成二进制数据然后进行跨进程传递。它能够做出这个类定义在从对象转换成二进制和二级制转换回对象的过程中不会改变的简单化假定。作为结果，`Parcelable`在Android的跨进程使用方面比`Serializable`快。

如果你想要的话，你是欢迎在你自己的类上实现`Parcelable`的。除此之外，所有你在Adnroid Java文档中看到实现了`Parcelable`的类都能放进`Intent extras`中。所以，例如，Uri实现了`Parcelable`，那么你就能把Uri放进
`Intent extras`。不是Android SDK的所有东西都是`Parcelable`的，但是一些像Uri的关键是`Parcelable`的。


更多`Parcelable`的细节，包括你如何在你的类上实现它，会在这本书之后内容出现。






