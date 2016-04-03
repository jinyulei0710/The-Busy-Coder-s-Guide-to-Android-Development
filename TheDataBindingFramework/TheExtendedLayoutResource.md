###扩展的布局资源

正如我们在前个例子中看到的，大部分我们传授给我们应用来运转数据绑定的知识是以扩展的布局资源语句的形式。根<layout>元素的最后一个字节点是我们布局文件以前持有的：处于这个布局视图层根部的视图或视图组。`<layout>`的其它孩子配置数据绑定行为(可能是将来的其它属性)。

有这样的想法之后，让我们探索下更多关于你能使用`<layout>`中元素做些什么。

####导入和静态

之前的样例相对于开始的样例缺少了一个特性：在标题中处理HTML。虽然Stack Overflow没有在问题标题中提供HTML标签，但是它的确是在问题标题中提供了HTML实体。一个“Foo&Bar”问题标题在我们的JSON中会是以“Foo & Bar”。网络访问那一章的样例通过Html.fromHtml()进行处理。但是，在我们数据绑定这章中却没有。

解决这个问题的一个方法是在Item中添加getter样式的方法，它会返回通过Html.fromHtml()处理的标题。例如，我们可以有一个getInterpretedTitle()方法或者getTitleWithEntitiesFixed()或getTitleAfterHavingRunItThroughHtmlFromHtml()。我们会在我们android:text表达式中引用这个方法(例如，@{item.interpretedTitle})。

但是，这模糊了模型对象和UI层的职责分割线。模型本身并不关心标题中有HTML实体，并且某些方式可能特有地需要这些HTML实体。转化这些HTML实体是UI职责，因为UI选择使用TextView,而TextView不会自动处理这些实体。

一个相对简单让我们的Html.fromHtml()逻辑回来的方法是把它应用到布局资源本身。例如，如果我们可以让我们的表达式变成@{Html.fromHtml(item.titile)}就会看起来很酷。

但是，如果你仅仅是实用这个语句而没有做其它改变，数据绑定会抱怨它不知道Html是什么。实际上，我们需要教导布局资源从哪导入Html。

为此，我们需要添加<import type="android.text.Html"/>到我们布局资源的<data>元素中。现在，生成代码会包含import声明并且我们对Html的引用也会得到解决。

你可以在[DataBinding/Static](https://github.com/jinyulei0710/cw-omnibus/tree/master/DataBinding/Static)样例代码中看到这个。这个样例项目是DataBinding/Basic的一个拷贝并作了两处修改（应用表达式和<import>），然后给了我们如下的布局资源:

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

    <import type="android.text.Html"/>

    <variable
      name="item"
      type="com.commonsware.android.databind.basic.Item"/>
    </data>

    <LinearLayout
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:orientation="horizontal">

    <ImageView
      android:id="@+id/icon"
      android:layout_width="@dimen/icon"
      android:layout_height="@dimen/icon"
      android:layout_gravity="center_vertical"
      android:contentDescription="@string/icon"
      android:padding="8dip"/>

    <TextView
      android:id="@+id/title"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="left|center_vertical"
      android:text="@{Html.fromHtml(item.title)}"
      android:textSize="20sp"/>

    </LinearLayout>
    </layout>

如果你运行这个版本的应用，并且最近问题里面恰好有一个问题的在它的标题中带着HTML实体，你会看到实体在ListView中是合适展示的。另一方面，如果你运行之前的样例，HTML实体会以HTML源形式显示(例如&amp而不是&)。

导入的规则令人回想起常规Java中的导入:

* 不要有冲突的导入(例如,android.view.Menu和com.myrestaurant.Menu)
* 不要尝试导入已经自动导入的(例如，java.lang.String)

####变量

正如我们在之前样例中看到的,你可以用<variable>元素代表可以被绑定表达式引用的对象。

用于<variable>元素的type属性可以是:

* 一个完全限定的类名，就如样例中的item变量
* 你通过<import>元素添加的类型
* 自动导入的所有Java类名

所以，在样例中，不是使用:
	
	<data>
	<variable
	   name="item"
	   type="com.commonsware.android.databind.basic.Item"/>
	 </data>
而是

	<data>
	 <import type="com.commonsware.android.databind.basic.Item"/>
	 <variable
	   name="item"
	   type="Item"/>
	 </data>  	 

如果你在不同资源集中对于不同配置的相同布局有着不同的版本(例如，res/layout/和res/layout-land／)，你的<layout>元素需要在它们之间兼容。这特别适用于变量。如果你在一个版本的资源中定义了一个字符串类型的变量foo，你不可以在另一个版本的资源中把它定义成Restaurant。每个布局资源都有一个绑定类，生成所有这个资源的不同版本，并且这个类对相同变量不能有两个分开的，冲突的定义。




	
