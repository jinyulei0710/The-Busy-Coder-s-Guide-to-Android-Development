###网格

到目前为止，我们专注的是把模型数据集合的视觉展现在竖直滚动的列表中。在AdapterView家族中，给定的AdapterView子类有着特定的视觉表现（Listview是竖直滚动的,GridView是二维网格等等）。在使用Recyclerview的时候，布局管理器的选择决定了绝大部份的视觉展现，而且从列表切换到网格简单到只要改变一行代码。

尽管如此，关键的是取决于你要的是什么的，网格形势的RecyclerView会更复杂，只不过是因为你现在有了两个维度来进行配置的权利。

####样例网格

要让RecyclerView使用网格是把LinearLayoutManager换成GridLayoutManager的问题。在RecyclerView/Grid的样例项目中，你会看到CardRippleList3样例应用的一个拷贝，现在在其中的MainActivity的OnCreate（）方法中使用的是GridLayoutManager。


	@Override
	public void onCreate(Bundle icicle){
   	 super.onCreate(icicle);
    
    	setLayoutManager(new GridLayoutManager(this,2));
    	setAdapter(new IconicAdapter());
	}


GridLayoutManager把跨度和上下问，作为构造器的参数。在样例中，跨度等于列数：每个由RecyclerView.Adapter返回的项会进入一个单行单跨度的单元。

在我们的例子中，我们请求了两个跨度，结果就变成了两列。

在此例中，有一个真的有行有列的单元格组成的网格。因此，行的高度是有行中最高的单元格决定的。举个例子，“amet单元格”比它所需的高是因为同一行的“consectetuer”单元格。在这章的之后内容中，我们会检视另外一个选项，StaggeredGridLayoutManager，在里面单元格不必要整齐的排列。

####选择列的数量

如果我们旋转这个样例的屏幕，你会决定看起来好一点，因为它们是改变用途为列表样式行了。


但是，一些应用可能有很小的单元。此外，我们还要考虑平板，甚至可能电视。你可能需要给予屏幕尺寸和方向来决定垮度。

一种尝试是使用整型资源。你可以有一个有<integer>元素的res/values/ints.xml文件，给出整型的名称和值。你也可以有res/values-w600dp或其它资源变化，这样你就能给不同的屏幕尺寸提供不同的值。然后，在运行时，调用getResources().getInteger（）来获取给当前设置使用的资源的正确值，并在你的GridLayoutManager构造器中使用。现在你能通过提供给构造函数的跨度来控制存在多少列了。

另一种尝试是，Chiu-ki Chan提议的，创建一个RecyclerView的一个子类，在其中你提供了一个自定义属性给大致的列宽度。然后在你的子类的onMeasure()方法中，你可以计算出跨度的数量从而得出你想要的列宽度。

当然，另一种利用屏幕控件的方式是增大单元格。默认情况下，它们会均匀增大，因为每个单元格占据一个跨度，并且跨度是均匀大小的。但是，你可以通过附加一个GridLayoutManager.SpanSizeLookup到GridLayoutManager改变这个行为.GridLayoutManager是用于负责指示给位置的项应该占据网格的多少跨度的。我们会在这章之后内容进行调查它是怎么工作的。
