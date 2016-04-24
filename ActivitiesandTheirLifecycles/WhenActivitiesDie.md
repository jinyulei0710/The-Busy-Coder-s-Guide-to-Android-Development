###当Activity死亡的时候

那么，什么能除掉一个activity呢。什么能触发导致onDestory被调用的事件链呢？

首要的，当用户按下返回按钮的时候，前台activity会被摧毁，在用户的导航流中控制权会返回到
前一个activity。

你可以从activity调用finish()来完成同样的事情。这主要是因为一些其他UI操作会指示用户
离开activity。但是，请不要人为地添加你自己的“exit”,"quit"或其他的菜单项或按钮到你的
activity上，只允许用户使用常规的Android导航选项,例如返回按钮。

如果你的activity没有一个在前台了，你的应用进程是被终止进而释放内存的候选者。就像之前提及到的，在这些情形中，Android可能会调用也可能不会调用onDestory()方法（onPause()和onStop()会在你的activity离开前台的时候被调用）。

如果用户导致设备经历了一次“配置改变，例如横竖屏切换”，Android的默认行为是摧毁你当前的前台activity
然后在它的位置创建一个崭新的activity。我们会在稍后一章中涵盖到这个内容。

并且，如果你的activity有一个没有处理的异常，你的activity会被摧毁，尽管如此Android不会在activity上调用任何
方法，因为它认为你的activity处于不稳定状态中。

