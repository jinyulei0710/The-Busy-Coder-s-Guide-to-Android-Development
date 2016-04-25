###不可见Activity的情形

有时候，你会要一个没有UI的activity。

这是相当少见见的。大部分时候，它会是需要你有一个activity，但是没有真正要以传统的activity样式UI展示给用户的。

例如，主界面启动按钮只启动activity。但是，你可能你需要一个主界面启动器仅仅触发一些在后台完成的工作，可能会使用
一个service(就像这本书稍后内容会讨论的)。

你有两种设置不可见activity的方式，包括在<activity>元素上使用一个特定的android:theme值。

最高效的选择是使用Theme.NoDisplay。有了这个值，就设置UI而言就没有做任何工作。但是，关键限制是所有的工作
需要在onCreate()方法中完成，并且在那里你需要调用finish()去触发让activity被摧毁。多数情况下，这就能很好地工作。

有时候，你需要一个不可见的activity挂起几秒钟，可能是在他被摧毁之前，等待一些回调结果。使用Theme.NoDisplay仍然能够工作...但是只是在较老的Android设备上。在Android 6.0以及更高的版本上，使用Theme.Nodisplay而没有在onCreate()中调用finish()（或者，技术上来说，在onResume()之前）会使你的应用崩溃。

变通方案是使用Theme.Translucent.NoTitleBar。这个实际上分配了一个UI给你，但是把它设置成了一个透明的背景并且没有动作栏。用户可能感知到activity在那里-例如，它出现在概览屏上(又名最近任务列表)。也因为activity真的在那，用户可能不能和它所看到的进行交互，例如在底层的主屏幕上。但是，如果activity本身能很快地finish()自己，并且是和用户实时交互的(例如，展示展示一些系统对话框)，你可能能够使用这个方法。
