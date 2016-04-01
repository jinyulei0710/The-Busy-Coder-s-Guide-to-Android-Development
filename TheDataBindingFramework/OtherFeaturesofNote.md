###其它需要注意的属性特征

还有很多你可以在数据绑定中应用的花哨功能。

####通过绑定类获取视图

样例应用已经从RowBinding中取回了用于行的ImageView控件。布局文件中所有的视图都有一个android:id值都对应着一个绑定生成类中的域。所以，像Picasso这样的情形，我们不能使用数据绑定来填充Imageview并必须采取传统的绑定它到适配器的逻辑，我们不需要自己做findViewById()调用。取而代之的，我们仅仅访问绑定类中的域就可以了。

####操作绑定中的变量

我们已经看过生成绑定类使用一个setter方法来把一个对象绑定到一个布局上。在样例应用中，我们已经调用了setItem()或setQuestion()来提供模型对象在绑定表达式中使用。如果需要的话，还有对应的getter方法（getItem(),getQuestion())来取回最后设置的值。

####视图,设置方法，和绑定

我们已经看过在android:text中使用绑定表达式，来为TextView设置文本。

实际上发生的事：

* 绑定系统评估表达式。着不仅给予我们要绑定的值也决定了值的数据类型。
* 数据绑定系统查找叫做set...（）的设置方法，其中...部分是基于属性名的（减去所有的命名空间），其中数据类型对应着表达式结果的数据类型。所以，在绑定表达式为android:text属性生成一个字符串的时候，数据绑定系统会在空间上查找setText(String)方法，在我们的情况中就是一个TextView。如果绑定表达式返回的是int,作为替代，数据绑定系统会查找setText(int)方法。在TextView的情形中，这个就存在，并且它预期int是一个字符串资源。这就是为什么，在Scored样例应用中，我们需要把整型转化为字符串。

当然，这仅仅是个简单的情形。

#####合成属性

数据绑定系统把属性名映射到setters。但是，如果你使用的属性名事实上不存在呢？

所有数据绑定系统做的事情是使用属性名来尝试找到一个相关的setter方法。属性名是不是LayoutInfalter支持的XML的结构是无关重要的。

这意味着你可以使用任意属性来映射到一个setter方法。

例如,ViewPager除了它从View或ViewGroup继承来的属性之外，没有它自己的XML属性。但是，你是欢迎使用像app:currentItem或app:pageMargin在你数据绑定增强的布局资源的（其中app指向一个你的自定义命名空间)。LayoutInfalter会分析它们，但是ViewPager会忽略它们。但是，数据绑定系统会高兴地让你绑定值到它们上，分别触发对setCurrentItem()和setPageMargin()的调用。

因此，不要感觉你是被限定只能使用这些由LayoutInflter和控件官方支持的属性。如果数据绑定系统可以找到一个setter,你就能使用它。

但是，这些合成属性有一个关键的限制，值必须是一个绑定表达式。这是真的，即使你没有真正地很好地进行表达式评估。

例如，这个就没有用:

	<ImageView
	 android:id="@+id/icon"
	 android:layout_width="@dimen/icon"
	 android:layout_height="@dimen/icon"
	 android:layout_gravity="center_vertical"
	 android:contentDescription="@string/icon"
	 android:padding="8dip"
	 app:error="@drawable/owner_error"
	 app:imageUrl="@{question.owner.profileImage}"
	 app:placeholder="@drawable/owner_placeholder"/>
	 
这里，我们有三个合成属性，app:error,app:imageUrl,和app:placeholder。只有app:imageUrl是使用了一个绑定表达式，并且它的使用是说的过去的，因为我们是从一个变量（question）拉取数据的。其它两个指向图片。按照理想的话，这个会有用。实际上，它并没有用，因为绑定系统忽略了这个属性，并且Android会投诉属性没有识别。

但是，这个就有用了:

	<ImageView
 		android:id="@+id/icon"
 		android:layout_width="@dimen/icon"
 		android:layout_height="@dimen/icon"
 		android:layout_gravity="center_vertical"
		 android:contentDescription="@string/icon"
 		android:padding="8dip"
 		app:error="@{@drawable/owner_error}"
 		app:imageUrl="@{question.owner.profileImage}"
 		app:placeholder="@{@drawable/owner_placeholder}"/>
 
现在,app:error和app:placeholer使用了绑定表达式...这个表达式仅仅返回了一个drawable资源引用。如果下面满足下面条件之一，这就会有用：
	 
1.在ImageView上存在给这些属性的setter方法(例如,setError()),在这个情形中，并没有,或者

2.我们使用其它的技术告诉数据绑定系统这些属性路由到了其它地方，就如在接下来两节会看到的

####使用不同的方法

####TODO


