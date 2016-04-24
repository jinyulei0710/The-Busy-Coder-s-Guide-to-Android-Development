###你的Activity的生死

当activity在如上四个状态之间切换的时候，Android会在你的activity中进行回调。

注意对于所有这些，你应该向上链接并调用父类的这个方法的版本，或者Android会抛出一个异常。


####onCreate()和onDestory()

我们已经在我们所有的例子中的Activity子类中实现了onCreate()方法。这个方法会在以下两个主要情形中
被带调用：

* 当activity首次被启动(例如，由于系统重启)，onCreate()被调用的时候会传入null参数。
* 如果activity已经处于运行状态了并且你的activity设置成基于不同的设备状态会有不同的资源（例如，
   水平对竖直）,你的activity会被重新创建并且onCreate()会被调用。我们会在这本书稍后内容更加详细地
   讨论。
   
不管activity如何被使用，这里是你初始化你的用户界面并设置所有需要立即完成的事情的地方。

在生命周期的另一端，当activity被关闭的时候onDestroy()可能会被调用，比如因为activity调用了
finish()方法(这个方法会结束activity)或者用户按下了返回键。因此，onDestory()方法是用阿里清理和释放
你从onCreate()方法中获取的资源，此外要确保所有你在生命周期方法之外启动的东西被中断了，例如后台线程。

记住，尽管如此，onDestory()方法可能不被调用。这可能在一些情形下发生：

* 你的应用因没有处理的异常崩溃了
* 用户强制中断了你的应用，例如通过设置应用。
* Android有着释放内存的迫切需求（例如，处理一个打进来的电话),想要中止你的进程，并且没有时间去调用所有的
  生命周期方法。
  
因此，onDestory()很有可能被调用，但是不能保证的。


此外，记住onDestory()的调用可能花费一段很长的时间。如果用户按下返回按钮来结束前台activity，onDestory()很快就会调用。但是如果，用户按下HOME按钮来到了主界面，你的activity不会直接被摧毁。直到Android决定优雅地结束你的进程的时候，onDestory()才会被调用。并且这个时间可能是几秒钟，几分钟，甚至几个小时。

####onStart(),onRestart(),和onStop()

一个activity能够来到前台，要么是因为它是首次被启动的，或者是因为它被遮盖之后被带回到了前台(
例如，被另一个activity，被一个打进来的电话)。

onStart()方法会在如上两种情形中被调用。onRestart()方法会在activity已经被中断然后重新启动的时候
被调用。

反过来说，onStop()在activity将要中断的时候被调用。它同样可能没有被调用，和onDestory()没有被调用的原因一样。
但是，onStop()通常在activity不可见之后相当快的时间内就被调用了，所以onStop()被调用的可能性比onDestory()还要高。


####onPause()和onResume()

onResume()方法在你的activity来到前台之前被调用，要么在初始化启动之后，要么是从终止状态
重新启动，要么在来电被清空之后。这是在你的activity上，刷新UI的一个绝佳地方。例如，你正在从一个服务拉取
一些改变信息（例如订阅的新条目)，onResume()同样是一个刷新当前视图的很好的时机，如果适用的话，启动一个后台线程
去更新视图(例如，通过一个Handler)。

反过来说，所有接管用户输入的-大部分是在另一个activity的驱动下-会导致你的onPause()被调用。在这里，
你应该撤销所有你在onResume()中所做的事情,例如终止后台线程，释放所有你获取的独占资源（例如，相机),和诸如此类的。

一旦onPause()被调用之后，Android具有了在任意时间杀死你的activity的进程的权利。因此，你应该不能指望接收
更多的事件。

那么，所以onPause()和onStop()直接的区别是什么呢？如果一个activity来到前台并充满了屏幕，
你当前的前台activity的onPause()和onStop()会被调用。但是如果，一个activity来到前台但是没有充满
屏幕，只有你当前的前台activity的onPause()会调用,因为它仍然是可见的。

####凑成对

如果你在onCreate()中初始化了某些东西，在onDestory()中把它清理掉。

如果你在onStart()中初始化了某些东西，在onStop()中把它清理掉。

如果你在onResume()中初始化了某些东西，在onPause()中把它清理掉。

换句话说，凑成对。例如，不要在onStart()中初始化了某些东西然后尝试在onPause()中进行清理，很多情形下onPause()可能被一系列地多次调用（例如，用户可能启动一个非全屏的activity，这会触发onPause()但没有触发onStop()和onStart()）。

你要根据你的需要选择生命周期对方法。你可能决定你需要两对(例如，onCreate()/onDestory()和onResume()/onPause()).只要别搞混它们之间对应关系。










