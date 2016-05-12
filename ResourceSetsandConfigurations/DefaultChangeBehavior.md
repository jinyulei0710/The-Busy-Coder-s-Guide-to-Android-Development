###默认的变化行为

当你在Android SDK调用加载资源的方法(例如，前面提到的setContentView(R.layout.main)),
Android会走查这些资源集，找到对于要求的来说正确的资源，并使用它。

但是如果配置改变发生在我们请求资源之后呢？例如，如果之前用户是竖着握着他们的设备的，然后旋转
到横屏？如果-land版本存在的话，我们会要一个布局的-land版本。并且，因为我们已经请求过资源了，
除了强迫我们再次请求这些资源之外，Android没有在运行中修改资源的方法。

所以，当运行中配置改变了，这就是默认情况下Android对我们的前台activity所做的事情。

####销毁并重建Activity
Android所做的最重大的事情是销毁并重建我们的activity。换句话说：

* Android在我们最初的activity实例上调用了onPause(),onStop()以及onDestroy()
* Android使用相同的用来创建最初实例的Intent来创建一个崭新的相同的activity类实例
* Android在新的activity实例上调用onCreate(),onStart()以及onResume()
* 新的activity出现在屏幕上

这可能看起来是侵入性的。你可能不希望Android仅仅因为用户抖了抖手腕并旋转了手机的屏幕，
就擦除一个非常好的activity。但是这是Android能保证重新请求所有我们资源的唯一方式。

####重建Fragment

如果你的activity使用的是fragment,新的activity实例会包含旧的activity实例
相同的fragment。其中包含静态的和动态的fragment。

默认情况下，Android会销毁并重建fragment，就像他销毁和重建activity一样。
但是，正如我们看到的，我们有一个告诉Android为它们保留某个动态实例的选项，
它会使用被旧的activity所使用的相同的activity，而不是重新创建一个新的
实例。

####重建视图

不管Android有没有重新创建所有的fragment，它会调用所有fragment的onCreateView()(外加
在最初的一套fragment上调用onDestroy())。换句话说，为了把它们注入新的activity实例中，
Android重建了所有的控件和容器。

####保留一些控件状态

Android会持有我们activity和fragment中的一些控件的实例状态。大部分时候，它
所持有的是明显的用户可改变的状态,例如:

* 已经输入到Edittext的东西
* 像CheckBox或RadioButton这样的CompoundButton有没有被选中
* 等等

Android会从旧的activity实例中的控件收集这个信息，把数据搬运到新的activity实例，
然后更新新的一套控件使其有相同的状态。

但是：

* 控件需要有一个ID来保存它们的状态。如果你是从一个布局资源中inflate出这个控件，
并且这个控件有android:id值，你就满足要求了。但是，如果你直接在Java代码中创建控件，
这些控件是没有ID的。你会需要通过调用setId()来给予它们一个ID或自己管理状态(使用之后
会在这章描述到的onSaveInstanceState()技术)。

* ID值需要时唯一的。如果有很多控件有着相同的ID,你就会遇到问题。通常，我们出现有多个
相同ID的控件的情形是这些控件是来自一个适配器的时候，例如一个ArrayAdapter。
并且，通常来说这些控件是只读的并且没有任何要保存的值。但是，如果你尝试
通过适配器把用户可修改的控件放入到布局中，否则你会有着有个有着相同ID的可修改控件，
你会需要自己进行管理(再一次的，使用之后会在这章描述到的onSaveInstanceState()技术)
