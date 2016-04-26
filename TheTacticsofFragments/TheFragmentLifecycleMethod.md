###Fragment生命周期方法

Fragment就像activity一样也有多个生命周期方法。实际上，它们所支持的生命周期方法大部分是和
activity一样的：

* onCreate()
* onStart()(但没有onRestart())
* onResume()
* onPause()
* onStop()
* onDestory()

大体上来说，就这些生命周期方法而言应用到fragment上的规则是和activity一样的。

除了这些和之前我们在这章分析的onCreateView()方法之外，你可以选择另外四个生命周期方法。

onAttach()首先会被调用，甚至在onCreate()之前，让你知道你的fragment已经被附加到一个activity上了。
你传入了持有你的fragment的Acticity。

onViewCreate()会在onCreateView()之后被调用。当你继承了onCreateView()实现但是需要配置结果视图的
时候，这就特别有用了，例如一个ListFragment需要设置一个适配器。

onActivityCreated()会在onCreate()和onCreateView()之后被调用，指示activity的onCreate()操作已经完成了。
如果你需要在你的fragment中进行一些以activity的onCreate()已经完成它的工作为前提的初始化工作，你可以使用onActivityCreate()用于这个初始化工作。

onDetach()是在onDestory()之后调用的，让你知道你的fragment已经和它的宿主activity解除关系了。

####onAttach()对onAttach()

如果你把你的项目的compileSdkVersion设置成23或者更高，当你尝试重载onAttach()的时候，你可能获得一个废弃的警告：

这是因为从API等级23开始，有两个版本的onAttach()（和onDetach()）。一个把activity作为参数，而另一个
把Context作为参数。

不同参数的onAttach()和onDetach()的职责是一样的：当fragment附加它的宿主或从它的宿主分离的时候让你知道。
但是，现在，宿主可以是任何继承了Context的东西，而不仅仅是activity。

尽管如此，在API等级22以及以下的版本，只有activity风格的onAttach()和onDetach()存在。
当你尝试到底如何为你的应用去解决这个问题的时候，这就变成了难题。

总的来说，如果你的minSdkVersion是低于23的，仅仅重载onAttach(Activity)是你的最好的选择。
它在所有支持fragment的设备上都是有效的。只重载onAttach(Context)就不会有效，因为较旧的设备会忽略它
(不顾Acticity是Context的子类)。你可以同时重载两个方法，但是在API等级23+设备上，两种风格的都会被带哦用，
这对你的Fragment子类来说可能不是个好办法。




