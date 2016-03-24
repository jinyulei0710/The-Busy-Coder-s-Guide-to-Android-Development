###一个普通的列表

回到最初的AdapterView和adapter的章节，我们有Selection/Dynamic这个样例应用。这个应用展示了25个拉丁单词的列表，每个都有单词的长度和附加的图标（长短单词是不一样的):

![wizard](/the_busy_coder's_guide/img/figure_435.png)

这呢，我们会回顾RecyclerView/SimpleList的样例项目，在移植Selection/Dynamic到使用RecyclerView的第一关。

####依赖

任何项目想要使用RecyclerView需要是哦那个来自Android支持包的recyclerview-v7库。Android Studio的用户仅仅需要在dependencies的闭包中的顶部加上它：

	dependencies{
   	compile 'com.android.support:recyclerview-v7:22.2.0'
	}

Eclipse用户会需要把类库项目从AndroidSDK中复制到另一个地方，导入这个拷贝到它们的Eclipse工作空间，然后把它作为类库添加到需要RecyclerView的应用。
这个过程在这本书之前的内容涵盖过。

####一个RecyclerViewActivity

ListView,我们可以使用ListActivity，来隔离一些ListView管理代码。在recyclerview-v7包中没有RecyclerViewActivity，但是我们可以创建一个:

	package cn.jinyulei.recyclerviewdemo;

	import android.support.v7.app.AppCompatActivity;
	import android.support.v7.widget.RecyclerView;

	public class RecyclerViewActivity extends AppCompatActivity {
    private RecyclerView rv = null;

    public void setAdapter(RecyclerView.Adapter adapter) {
        getRecyclerView().setAdapter(adapter);
    }

    public RecyclerView.Adapter getAdapter() {
        return (getRecyclerView().getAdapter());
    }

    public void setLayoutManager(RecyclerView.LayoutManager mgr) {
        getRecyclerView().setLayoutManager(mgr);
    }

    public RecyclerView getRecyclerView() {
        if (rv == null) {
            rv = new RecyclerView(this);
            rv.setHasFixedSize(true);
            setContentView(rv);
        }
        return (rv);
    }

	}

重要的部分是getRecyclerView（）方法。这里如果我们没有初始化RecyclerView,我们创建它的一个实例然后通过setContentView（）设置活动的内容。我们在RecyclerView上调用了setHasFixed(true)，告诉它它的大小不应该基于适配器的内容有所改变。知道这个能帮助RecyclerView更高效工作。

RecyclerViewActivty也有与它们的ListActivity类似的getAdapter和setAdapter()。我们会在这节之后的内容中探究适配器中有什么不同。我们也有一个setLayoutManager()的便利方法，它只是调用了底层的setLayoutManager()-我们在会下一节的RecyclerView的上下文了解布局管理器是什么。

ListActivity中还有其它的特性没有在在这里的RecyclerViewActivity反应出来，仅仅使RecyclerViewActivity简短。值得注意的，ListActivity支持包含ListView的自定义布局或者创建它自己的。RecyclerViewActivity尽管并不支持这个，但做一些较小的扩展就可以了。


####(布局管理器)LayoutManager

项目的真正活动是MainActivity,它包含了单个方法：onCreate()

	@Override
	public void onCreate(Bundle icicle){
  	 super.onCreate(icicle);
   
  	 setLayoutManager(new LinearLayoutManager(this));
  	 setAdapter(new IconicAdapter());
	}


在链接到父类之后，我们第一件要做的是调用setLayoutManager()，它会把一个RecyclerView.LayoutManager跟我们的RecyclerView联系在一起。具体来说，我们使用了一个LinearLayoutManager。

Listview有着竖直滚动的行融入它的实现。相似的，GridView也有着平面的竖直滚动网格融入它的实现。RecycView,在另一方面，完全不知道如何放置它的孩子。这个工作被委托给了一个RecyclerView.LayoutManager，所以不同的方法可以在需要的时候插入。

recyclerview-v7装载的RecyclerView.LayoutManager基类有着三个具体的子类：
* LinearLayoutManager，它实现了一个竖直滚动的列表，类似于Listview
* GridLayoutManager，它实现了一个平面的竖直滚动列表，类似于GridView
* StaggeredGridLayoutManager,它实现了一个“交错的网格”，它有着像GridView的单元格，但是cells并不是一个大小的。

此外，创建你自己的RecyclerView.LayoutManager是特别可能的，或者使用来自第三方库的。

尽管如此，在此例中，我们继续使用一个简单的LinearLayoutManager，因为我们在尝试复现ListView的功能。

####适配器

我们的onCreate()方法也调用了setAdapter(),把一个RecyclerView.Adapter和我们的RecyclerView（具体的，初始的Selection/Dynamic样例应用中的一个修改版）。如同AdapterView家族一样，RecyclerView使用一个适配器帮助把我们的数据模型转换成视觉表现。但是，Recycler.Adapter是跟Listview或GridView使用的ListAdapter有很大不同的。

怀旧的ArrayAdapter，一个RecyclerView.Adapter使用的泛型，并且声明我们要适配的东西类型。但是，ArrayAdapter使用泛型来描述模型数据。RecyclerView.Adapter换而使用泛型来识别ViewHolder,它会负责做真正绑定模型数据到行布局的工作。

	class IconicAdapter extends RecyclerView.Adapter<RowHolder> {


        @Override
        public RowHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return (new RowHolder(getLayoutInflater().inflate(R.layout.row, parent, false)));
        }

        @Override
        public void onBindViewHolder(RowHolder holder, int position) {
            holder.bindModel(items[position]);
        }

        @Override
        public int getItemCount() {
            return (items.length);
        }
    }

在我们的情况中，IconicAdapter使用的是一个RowHolder类我们会在下一节检视。

一个RecyclerView.Adapter有着三个需要实现的抽象方法。

一个是getItemCount（）,它充当getCount和ListAdapter中的关系一样，指示RecyclerView中会有多少个项。在IconicAdapter的情况中，这是基于静态String对象数组的长度的，就跟它和IconicAdaper在Selection/Dynamic样例应用中的关系一样：

 	private static final String[] items = {
            "lorem", "ipsum", "dolor",
            "sit", "amet",
            "consectetuer", "adipiscing", "elit", "morbi", "vel",
            "ligula", "vitae", "arcu", "aliquet", "mollis",
            "etiam", "vel", "erat", "placerat", "ante",
            "porttitor", "sodales", "pellentesque", "augue", "purus"
   	 };


其它两个方法是onCreateViewHolder()和onBindViewHolder（）。这些有点让人回想起被CursorAdapter使用的newView()和bindView()方法。但是，比起直接和视图打交道，onCreateViewHolder（）和onBindViewHolder()打交道的ViewHolder对象，就和最初在selection控件那章看到的视图持有者（View Holder）的形式一样。

onCreateViewHolder(),就像名字所启示的，需要创建，配置并返回一个视图持有者给我们列表的某一行。它传了两个参数:

* 一个是ViewGroup，它持有被持有者管理的视图，大多数是是是用layout inflation
* 一个是int，在我们有多种视图类型的情况时，它是我们所使用的某个视图类型

IconicAdapter实现inflate我们的行视图(R.layout.row)并且把它传递给RowHolder构造器，返回
结果RowHolder.

onbindViewHolder()负责基于数据模型更新某个位置的ViewHolder。IconicAdpter通过吧模型传递到一个私有的bindModel（）方面来实现RowHolder.

RecyclerView.Adapter还有很多你能重载的方法，我们会在这章的后面内容看到一些。但是对于简单的列表来说，这三个就足够了。

####ViewHolder

RecyclerView负责把我们所需的来自模型的数据绑定到列表的行控件中：

	 static class RowHolder extends RecyclerView.ViewHolder {
        TextView label = null;
        TextView size = null;
        ImageView icon = null;
        String template = null;

        public RowHolder(View row) {
            super(row);
            label = (TextView) row.findViewById(R.id.label);
            size = (TextView) row.findViewById(R.id.size);
            icon = (ImageView) row.findViewById(R.id.icon);
            template = size.getContext().getString(R.string.size_template);

        }

        void bindModel(String item) {
            label.setText(item);
            size.setText(String.format(template, item.length()));

            if (item.length() > 4) {
                icon.setImageResource(R.drawable.delete);
            } else {
                icon.setImageResource(R.drawable.ok);
            }
        }

    }

但是，除了需要使用RecyclerView.ViewHolder的基础类。适配器和viewholder之间就没有其它某个协议了。你可以创造你自己的API。这里呢，我们使用了RowHolder构造器传入了行视图，在rowholder中构建器取得单个控件然后设置我们的字符串资源模版。然后，私有的bindModel（）方法获取我们的模型对象（一个字符串）然后把它绑定到行的控件上，随即实现了我们的业务规则。

####结果

就像这个项目名字所示，给出了我们一个简单的列表:

![wizard](/the_busy_coder's_guide/img/figure_434.png)

和Listview一样，RecyclerView（使用RecyclerView.LayoutManager）通过可用的行处理竖直滚动。

####少了什么?

但是，比起使用Listview的selction／Dynamic版本缺少了两个东西。

首先，行之间没有分割线。这对这个特定行布局可能不是个大问题，但是对需要视觉分割的布局来说就是了。
我们会在下一节探索实现的方式。

第二个，我们缺失了点击事件。用户可以尽他所希望的点击行。不仅用户通过这些点击不能得到任何视觉反馈，而且我们没有setOnItemClickListender（）找出这些点击。我们会在这章之后内容去弥补这个缺口。

RecyclerView也缺少一些其它我们可以从ListView获取的东西，这些没有在这个例子中使用，例如:
* 选择模式，选择列表
* header和footer视图
* 选中行的概念
* 过滤支持
* 等等

在本章中，我们会探索其中一些问题以及如何解决它们。
