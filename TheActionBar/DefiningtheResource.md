###定义资源

获取工具栏图标和动作溢出项到动作栏的最简单方式是以使用XML资源菜单的形式。由于历史原因这被称作“menu”资源，因为这些资源最初是被像选项菜单所使用的。

你可以添加一个res/menu/目录到你的项目中，并把菜单XML资源放进去。

在AndroidStudio中，你需要直接编辑XML文件。以下是来自ActionBar/ActionBarDemoNative的
res/menu/actions.xml:

	<?xml version="1.0" encoding="utf-8"?>
    <menu xmlns:android="http://schemas.android.com/apk/res/android">

	<item
		android:id="@+id/add"
		android:icon="@drawable/ic_action_new"
		android:showAsAction="always"
		android:title="@string/add"/>
	<item
		android:id="@+id/reset"
		android:icon="@drawable/ic_action_refresh"
		android:showAsAction="always|withText"
		android:title="@string/reset"/>
	<item
		android:id="@+id/about"
		android:icon="@drawable/ic_action_about"
		android:showAsAction="never"
		android:title="@string/about">
	</item>

     </menu>

每个菜单项(XML中的<item>元素)有着四个你会要配置的东西：

1.标识。这会创建一个新的R.id值，这个值会与菜单项相关，就和R.id和我们布局中的控件相关差不多。
当用户点击我们工具栏按钮或动作溢出栏的一项或者我们会使用这个ID来决定。

2.标题。如果这个项出现在动作溢出菜单中，或者作为他的工具栏按钮的可选部分，这个文本会出现。同样的，如果用户长按图标，这个标题会会以提示信息的形式出现在动作项之上。代表性的，你会使用一个字符串资源引用(例如，@string/add)，为了更好地支持国际化。

3.图标。如果你的像以工具栏按钮的形式出现，这个图标会被按钮使用。

4.标记指示这个项改如何描绘在动作栏中。你可以让它一直是一个工具栏按钮，在空间足够的情况下是工具栏按钮。你也可以选择附加|withText到always或ifRoom上，来指示你想要工具栏按钮同时有着图标和标题，而不仅仅是图标。注意，always是不能确保的-如果你要求100个always项的话，你不会有足够容纳它们的空间。但是，always项对动作栏空间有着比ifRoom项有着更高的优先级。







