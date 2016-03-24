###进击的类库

在这个时候，你可能为了让RecyclerView能使用所做的大量工作，无法抑制你的愤怒并扯掉你的衣服。

（专业建议：不要在公共场合脱衣服，避免触犯法律）

毫无疑问RecyclerView是需要自己装配的典型代表。但是，其他开发者带着能弥补这些缺口的类库走到台前，让你不用全部自己去做。

注意作者没有尝试很多这些类库，并且列在这里的并不代表认可，也不是对没有列在这里的类库的打击。

####DynacmicRecyclerView

DynamicRecyclerView 类库提供了:

* 通过自定义的onItemTouchListener实现对项的拖放
* 通过自定义的onItemTouchListener实现水平滑动移除
* 类似这章展示的选择模式
* 使用另一个自定义的onItemTouchListener实现点击样式事件。

####Advanced RecyclerView

Advanced RecyclerView 类库提供了它自己的拖放以及滑动移除实现。
你在你的RecyclerView.Adapter和RecyclerView.ViewHolder类上实现了支持拖放或滑动移除的接口，外加使用管理类
进行支持绑定，而不是使用OnItemTouchListener实现。

它也支持展开项，点击项时展开或收拢位于下方的视图，提供作为ExpanableListView的替代尝试。并且，它有着一些item装饰，
包括基本的分隔线实现。

####SuperRecyclerView

SuperRecyclerView类库有着它自己的滑动移除实现。它也支持“swipe layout”模式，当水平滑动出现的时候，去除项上的一系列上下文操作。

它也提供了:

* scrollbars(RecyclerView没有的)
* 处理数据异步加载的进度条和空视图
* 一个无休止的或无限滚动的实现，当用户滚动到数据底部的时候，你有着获取更多数据的机会
* sticky headers,当用户滚动的时候，最顶部的头牵制在RecyclerView的顶部，直到新的头划到顶部。

但是，这个类库需要你继承一个自定义的RecyclerView子类。

####FlexibleDivider

FlexibleDivider类库做的只有一件事，提供分隔线。但是，它提供了对分隔线的深度支持，使用它你可以很简单地控制
所有类型的样子，从颜色宽度到边距以及path效果(例如：虚线对比实线)。

RecyclerView/FlexDividerList样例项目是之前ManualDividerList样例的一个复制，只是现在的分隔线是由FlexibleDivider类库提供了，
这个类库是通过build.gardle文件加载的：

    dependencies{
      compile 'com.android.support:recyclerview-v7:22.2.0'
      compile 'com.yqritc:recyclerview-flexibledivider:1.0.1'
    }

 然后我们使用类库的HorizontalDividerItemDecotation来设置我们的分割线:
    
      @Override
      public void OnCreate(Bundle icicle){
        super.onCreate(icicle);
        setLayoutManager(new LinearLayoutManager(this));

        RecyclerView.ItemDecoration decpr=new HorizontalDiverItemDecoration.Builder(this)
        .color(getResouces().getColor(R.color.primary)).build();
        getRecyclerView().addItemDecoration(decor);
        setAdapter(new IconicAdapter());
      }   

结果是实心蓝色分隔线:

![wizard](/the_busy_coder's_guide/img/figure_449.png)

当然，通过这个类库，你可以通过他的建造者样式的API改变更多而不只是它的颜色。
