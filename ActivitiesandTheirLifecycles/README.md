##Activity们和它们的生命周期

一个Android应用会有多个互无关联的UI界面。例如，一个日历应用需要让用户浏览日历，
浏览单个事件的详情，编辑一个事件(包括添加一个新的)，等等。并且在较小屏设备上，像大部分的
手机，你可能没有空间把所有这些一次性地挤在屏幕中。

为了解决这个问题，你可以有多个activity。你的日历应用可能有一个展示日历的activity，一个用作
添加或编辑时间，一个用作提供日历如何工作的设置，一个用作网上求助，等等。这些activity中的一些可能是你应用私有的，
单同时其它的可能被第三方的启动，比如你的启动activity主屏幕上是可用的。

所有这些表明了你的多个activity中的其中一个有着启动其它activity的方法。例如，如果某人从查看日历的activity点击了一个事件，你可能想要为这个事件展示查看事件的activity。这就意味着，你需要以某种方式能够使查看事件的activity启动并且展示某一特定的事件（用户点击的那一个）。

这可以进一步分解成两种情形:

* 你知道你所要启动的activity，可能是因为它是你应用中的另一个activity。
* 你有着某个东西的引用（例如，一个网页)，并且你想要你的用户能用它做些什么，但是你不知道有什么可选的操作。

这两种情形在这章中都会涵盖到。

此外，时常对你来说知道什么时候activity来到和离开前台是很重要的，因为这样你就可以自动保存或刷新数据，等等。这就是所谓的“activity生命周期”,并且我们也会在这章中详细分析它。

目录：

* [创建你的第二个(第三个以及...)Activity](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/CreatingYourSecondActivity.md)
* [警告！包含显式意图](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/Warning!ContainsExplictIntents.md)
* [使用隐式意图](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/UsingImplictIntent.md)
* [Extra!Extra!](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/extra!extra!.md)
* [考虑`Parcelable`](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/PonderingParcelable.md)
* [同步和结果](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/AsynchronicityandResults.md)
* [薛定谔的Activity](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/Schroedinger'sActivity.md)
* [你的Activity的生死](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/Life%2CDeath%2CandYourActivity.md)
* [当Activity死亡的时候](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/WhenActivitiesDie.md)
* [漫步生命周期](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/WalkingThroughtheLifecycle.md)
* [回收Activity](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/RecyclingActivities.md)
* [应用：不只是Activity](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/ApplicationTranscedingtheActivity.md)
* [不可见Activity的情形](https://github.com/jinyulei0710/The-Busy-Coder-s-Guide-to-Android-Development/blob/master/ActivitiesandTheirLifecycles/TheCaseoftheInvisibleActivity.md)




