
##RecyclerView(垃圾回收视图)
视觉表现数据项是许多移动应用很重要的一方面。传统的Android的这项功能，是AdapterView控件家庭实现的。但是，它们有着它们的限制，特别是关于列表内容动画变化的高级功能。

在2014年，Google通过Android支持包发布了RecyclerView。开发者可以添加recyclerview-v7到它们的项目中并使用Recycler作为大部分AdapterView家族的替代品。RecyclerView是完全新的一个更灵活或的容器，有着很多的钩子和代理从而允许插入的行为。

这有着两个主要影响：

1.Recyclerview确实与其对应的Adapterview强大的多.

2.RecyclerView不能拿来就用，甚至于复现基础的ListView/GridView功能也要写相当多的代码。

在这章中，我们会从零开始学习RecyclerView,从基本操作开始。许多其它地方的ListView会在这复现，来看如何使用RecyclerView完成相同的事情。并且我们会探究一些增加的功能，这些功能或许能让RecyclerView在高端Android应用上值得为之付出努力。

###准备工作

为了理解这章需要你已经读了核心章节，特别是AdapterView和adapter那章。

一节包含了自定义XMLdrawables.另一节展示使用的内容是从媒体内容提供者拉取的。

这章也包含了像动作模式和其它的高级ListView技巧。

###AdapterView和它的不足

AdapterView，特别是它的子类ListView和GridView,在Android应用开发中承担了重要的工作。并且，对基本场景来说，它们还是相当令人满意的。

但是，也存在问题。

可能最大的策略问题是更新一个AdapterView倾向于要么全部更新要么不更新。如果模型数据改变了-新的行添加了，存在的行移除了或者数据改变,这些可能影响AdapterView的展现-唯一提供很好支持的解决方案是调用notifyDataSetChanged()并让AdapterView重新构建自己。这是缓慢的并且可能对选择状态有影响。并且如果你真的要在复杂的改变，并且在行添加或移除的时候使用动画效果，一半是不可能的。

策略上，AdapterView,AbsListView(ListView和GridView的直接父类)，和ListView对许多门外汉来说就是一大堆跟pasta一样的代码。这些类中堆了太多的责任，以至于对Google来说可维护性是一种挑战，扩展性就属于理想而并非事实了。

###了解垃圾回收视图
RecyclerView就是为纠正这些瑕疵而设计的。

RecyclerView它自己本身除了帮助管理视图回收（例如：竖直列表的行回收）之外做了很少的工作。它委托几乎所有其它的东西给其它的类，例如:

* 一个布局管理器，负责把视图组织成多样的结构（竖直列表，网格，交错网格，等等）
* item装饰，负责给视图应用效果和光定位，例如在竖直列表之间加上分隔线
* item动画，负责在模型数据改变的时候的动画效果

这是在adapters和view holders（常见AdpterView使用方法的优质标志）之上的。

因为想布局管理器是通过一个抽象类处理的并且把一些具体的实现放回去了，第三方开发者可以共享选择给开发者使用，就像Google做的。

另一方面，虽然RecyclerView没有像ListView或GridView那样立即可用。在recyclerview-v7库中，并不是所有的没了的东西都提供了，需要你要么自己写一捆代码要么依赖那些第三方库可以完成任何事。

回到最初的AdapterView和adapter的章节，我们有Selection/Dynamic这个样例应用。这个应用展示了25个拉丁单词的列表，每个都有单词的长度和附加的图标（长短单词是不一样的):

![wizard](/the_busy_coder's_guide/img/figure_433.png)

这呢，我们会回顾RecyclerView/SimpleList的样例项目，在移植Selection/Dynamic到使用RecyclerView的第一关。

任何项目想要使用RecyclerView需要是哦那个来自Android支持包的recyclerview-v7库。Android Studio的用户仅仅需要在dependencies的闭包中的顶级加上它：

	dependencies{
   	compile 'com.android.support:recyclerview-v7:22.2.0'
	}

Eclipse用户会需要把类库项目从AndroidSDK中复制到另一个地方，导入这个拷贝到它们的Eclipse工作空间，然后把它作为类库添加到需要RecyclerView的应用。这个过程在这本书之前的内容涵盖过。

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

重要的部分是getRecyclerView（）方法。这里如果我们没有初始化RecyclerView,我们创建它的一个实例然后通过setContentView（）设置活动的内容。中途，我们在RecyclerView上调用了setHasFixed(true)，告诉它它的大小不应该基于适配器的内容有所改变。知道这个能帮助RecyclerView更高效工作。

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

####分割线选项

在Recyclerview中主要有两种实现视觉分割的方法：

1.确保这是由布局自身处理的，例如使用CardView

2.使用RecyclerView.ItemDecorator实现项之间的常见分割线

两个技术方案都会在这章中涵盖到。

卡片是在移动开发中的一种流行的视觉隐喻。把内容集合分到卡片中使你如何组织内容来适应多种屏幕尺寸和方向变的清晰。一些情况下，你可能是单列的卡片，同时在其它情况下，你可能有横向排列的卡片。

在2014年，Google发布了cardview-v7,另一个在Android Support中的包。它所提供的是CardView。CradView是FrameLayout的一个简单子类，提供了卡片界面，包括圆角和落影。特别的是，Android5.0中CardView会基于控件高度使用的默认落影效果。之前的发布版本使用仿真的落影效果。这样的话，你就能得到向后兼容到API等级7的合理一致外观。

为了使用这个，你需要在的应用项目中添加cardview-v7库。AndroidStudio用户只需添加在Android支持仓库的cardview－v7 这个artifact就可以了，就像在RecyclerView/CardViewList这个样例项目中所看到的一样:


	dependencies{
   	compile "com.android.support:recyclerview-v7:22.2.0"
   	compile "com.android.support:cardview:22.2.0"
	}

Eclipse用户则需要与附加recyclerview－v7到它们项目中所使用的方式，做同样的是来附加cardview－v7。

然后，你就可以把你的行布局包装进CardView（更准确的说是，android.support.v7.widget.CardView）:

	<?xml version"1.0" encoding="utf-8"?>
	<android.support.v7.widget.CardView
 	xmlns:android="http://schemas.android.com/apk/res/android"
 	xmlns:cardview="http://schemas.android.com/apk/res-auto"
 	android:layout_width="match_parent"
 	android:layout_height="wrap_content"
 	android:layout_margin="4dp"
 	cardview:cardCornerRadius="4dp">
  	<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <ImageView
      android:id="@+id/icon"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_vertical"
      android:padding="2dip"
      android:src="@drawable/ok"
      android:contentDescription="@string/icon"/>

    <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

      <TextView
        android:id="@+id/label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="25sp"
        android:textStyle="bold"/>

      <TextView
        android:id="@+id/size"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="15sp"/>
    </LinearLayout>

 	 </LinearLayout>
	</android.support.v7.widget.CardView> 

没有更改原本RecyclerView/SimpleList样例的其它任何代码，我们得到了这个:

![wizard](/the_busy_coder's_guide/img/figure_435.png)

注意：落影效果可能在Android5.0＋模拟器上无法显现，特别是你已经关闭宿主GPU模式的时候。Cardview会好好的，只是没有了落影效果。

####手工的

CardView对你来说可能不是你列表的合适视觉方法。可能你需要一个常规的分割线，就像Listview中的那样。虽然是可能的，但是不是很直接。

RecyclerView把分割线当做是“item decorations”。有一个RecyclerView.ItemDecoration抽象类，你能对其进行扩展从而处理项装饰。并且你可以通过addItemDecoration（）添加到Recyclerview。如名所示，如果需要的话你可以有多个装饰。

但是，Google并没有提供任何具体的装饰的实现。

少数有进取新的开发者尝试了这个，得出了解决方案。RecyclerView/ManualDividerList样例项目展示了装饰的使用。

首先，我们需要可绘制的资源给分割线本身

	<?xml version="1.0" encoding="utf-8"?>
	<shape xmlns:android:"http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
	<size
   	android:width="1dp"
  	 android:height="1dp"/>  
	 <solid android:color="@color/divider"/>         


这是一个ShapeDrawable，在drawables这章涵盖到了。大的是一个solid填充，这里指向填充所使用的一个色彩资源：

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
 	  <color name="divider">#ffaaaaaa</color>
	</resources>


ShapeDrawable是一个1dp大小的正方形。实际上的话，在动态生成时会由装饰着充满recycler的宽度。

注意这个drawable没有任何特别之处。你可以有中间大两边小的斜线。或者，你可以使用.9PNG文件，Android5.0+上的矢量图，或者其它能很好调整大小的。

接下来，我们需要一个RecyclerView.ItemDecoration的实现类，例如样例项目中的HorizontalDividerDecoration:

	package com.commonsware.android.recyclerview.manualdivider;

	import android.graphics.Canvas;
	import android.graphics.drawable.Drawable;
	import android.support.v7.widget.RecyclerView;
	import android.view.View;

	// inspired by https://gist.github.com/polbins/e37206fbc444207c0e92

	public class HorizontalDividerItemDecoration extends RecyclerView.ItemDecoration {
 	 private Drawable divider;

  	public HorizontalDividerItemDecoration(Drawable divider) {
    this.divider=divider.mutate();
 	 }

  	@Override
 	 public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
   	 int left=parent.getPaddingLeft();
   	 int right=parent.getWidth()-parent.getPaddingRight();

   	 int childCount=parent.getChildCount();

   	 for (int i=0; i<childCount-1; i++) {
      View child=parent.getChildAt(i);
      RecyclerView.LayoutParams params=
          (RecyclerView.LayoutParams)child.getLayoutParams();

      int top=child.getBottom()+params.bottomMargin;
      int bottom=top+divider.getIntrinsicHeight();

      divider.setBounds(left, top, right, bottom);
      divider.draw(c);
   	 }
 	 }
	}


这个类把Drawable作为输入，所以如果需要的话它能给不同的分割线使用。HorizontalDividerItemDecoration调用了Dawable的mutate()方法来让获取一个Drawable，然后就能跟任何已存在的Drawable实例独立开来。这在使用Drawable资源的时候是很重要的，因为相同的资源可用被重用，所以改变核心Drawable本身是不安全的。

HorizontalDividerItemDecoration的主要逻辑存在于onDrawOver()方法中。它会被调用让我们在RecyclerView上画出来。这里我们做了以下的事情：

* 相对于RecyclerView的左右边缘，决定左右需要画的宽度，但是要减去内边距，因此我们只能在内边距内画。
* 遍历RecyclerView中的孩子视图，找到分割线的竖直位置，调整分割线来适应想要的空间，然后在所提供的画布上画出分割线。

使用这样的小魔法，现在仅仅要做的是附加我们的HorizontalDividerItemDecoration到我们的RecyclerView,在MainActivty中的onCreate（）方法中完成。

	@Override
	public void onCreate(Bundle icicle){
   	  super.onCreate(icicle);
   	  setLayoutManager(new LinearLayoutManager(this))
     
    	 Drawable divider=getResources().getDrawable(R.drawable.item_divider);
     
     	getRecyclerView().addItemDecoration(new HorizontalDividerItemDecoration(divider));
     	setAdapter(new IconicAdapter());
	}

这个样例项目的其余部分从原本的这章开头的SimpleList样例项目复制过来就可以了。

结果是我们在子视图之间有了一个分割线：

![wizard](/the_busy_coder's_guide/img/figure_436.png)


如果必须自己来做所有这些事情让你感到烦躁，有能拿来就用的能提供item decotations的第三方库。我们会看这章之后的内容中审查一个。

####处理点击事件

但是，有个漂亮的分割线并没有解决更大的问题，响应输入。

这个Recyclerview本身，除了滚动没有做任何跟输入有关的事情。任何处理用户点击和触发某种类型的反馈是Recyclerview中的视图处理的,例如列表样式中的行。

这样有这样的好处。像RatingBar这样的可点击控件，在ListView的行中一直跟行本身的点击事件冲突。让行能点击的同时，也让行里的内容也能点击就有点棘手了。使用RecyclerView,你能更加显式地控制这类事情是如何处理的，因为你是那个设置所有on－click处理逻辑的人。

当然，这没有给用户很大帮助。用户并不关心什么代码负责输入。用户只想要提供输入。如果你展现给他们竖直滚动的列表样式的用户界面，他们会尝试在列表中的行进行点击病期待某种类型的输出。

RecyclerView的实现方式，意味着很大程度上你要依靠自己处理输入。这就需要更多的代码，在一个理想的世界，会提供一个马上就能用的recyclerview选项。

####响应点击

在它的核心，响应点击事件是设置一个OnClickListener到合适的视图上的问题。

所以，举个例子，RecyclerView／CardClickList样例项目是CardViewList样例的一个拷贝，在这个项目中我们在RecyclerView.ViewHolder中的行视图上调用了setOnClickListener()，现在重命名为RowController:

	package com.commonsware.android.recyclerview.cardclicklist;

	import android.support.v7.widget.RecyclerView;
	import android.view.View;
	import android.widget.ImageView;
	import android.widget.TextView;
	import android.widget.Toast;

	class RowController extends RecyclerView.ViewHolder
    	implements View.OnClickListener {
 		 TextView label=null;
 		 TextView size=null;
  		 ImageView icon=null;
 		 String template=null;

 	 	RowController(View row) {
   		 super(row);

    	label=(TextView)row.findViewById(R.id.label);
   		 size=(TextView)row.findViewById(R.id.size);
    	icon=(ImageView)row.findViewById(R.id.icon);

    	template=size.getContext().getString(R.string.size_template);

   		 row.setOnClickListener(this);
	  }

 	 	@Override
 	 	public void onClick(View v) {
   		 Toast.makeText(v.getContext(),
    	   	 String.format("Clicked on position %d", getAdapterPosition()),
    	    	Toast.LENGTH_SHORT).show();
  	}

  	void bindModel(String item) {
   	 	label.setText(item);
    	size.setText(String.format(template, item.length()));

    	if (item.length()>4) {
   	   		icon.setImageResource(R.drawable.delete);
    	}
    	else {
    	 	 icon.setImageResource(R.drawable.ok);
    	}
  		}
	}



* 在这个样例中，onClick（）方法做的事展示一个Toast。但是，你可以在事件总线引发一个事件，或者
* 在某一个提供的接口上调用方法（例如传入到RowController构造器）来委托事件到高阶的控制者，或者
* 不管什么其它可能需要的东西

在此例中，因为在行中的控件没有交互造成点击事件的消耗，用户可以在行的任意的地方进行点击，Toast都会出现。
如果你遇到更复杂的情形，例如核对清单中－你可以自己决定如何处理行中不同部分的点击事件。我们马上会在之后这章中看到清单了。

####点击的视觉效果

但是，如果当你运行CardClickList样例的时候，你会注意到一个主要的遗留瑕疵，没有点击事件的视觉反馈。Toast出现了，但是用户过去习惯于在行之上看到某种短暂的状态改变，例如颜色闪一下。又一次，我们有了控制我们认为适合的短暂状态改变的能力...我们就有责任让它发生。

有一些解决这个问题的方法，例如在这节中列出来的。

选项＃1:顶部的半透明选择器

一个方法是MarkAllision在他的stylingAndroid博客建议的模仿Listview可用的drawselectorOnTop方法。使用的是FrameLayout，你把一个半透明选择器去放到了行之上，在这个地方选择器实现了点击反馈。

RecyclerView/CardRippleList样例项目是CardClickList复制了CardClickList利用了Mr.Allison的方法。
修正的row.xml利用了CardView是FrameLayout子类这个事实，所以它放置了一个空白视图到LinearLayout上，行的核心内容所在。

	<?xml version="1.0" encoding="utf-8"?>
    <android.support.v7.widget.CardView
  	xmlns:android="http://schemas.android.com/apk/res/android"
  	xmlns:cardview="http://schemas.android.com/apk/res-auto"
  	android:layout_width="match_parent"
  	android:layout_height="wrap_content"
  	android:layout_margin="4dp"
  	cardview:cardCornerRadius="4dp">

  	<LinearLayout
   	 android:layout_width="match_parent"
    	android:layout_height="wrap_content"
    	android:orientation="horizontal">

   	 	<ImageView
     	 android:id="@+id/icon"
     	 android:layout_width="wrap_content"
      	 android:layout_height="wrap_content"
      	 android:layout_gravity="center_vertical"
      	android:padding="2dip"
     	 android:src="@drawable/ok"
      	android:contentDescription="@string/icon"/>

    <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

      <TextView
        android:id="@+id/label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="25sp"
        android:textStyle="bold"/>

      <TextView
        android:id="@+id/size"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="15sp"/>
    </LinearLayout>
     </LinearLayout>
     <View
     android:layout_width="match_parent"
      android:layout_height="match_parent"
   	 android:background="?android:attr/selectableItemBackground"/>
    </android.support.v7.widget.CardView>
    
    
视图的背景是当前主题的selectableItemBackground。在使用theme上的应用上，它会给你一个橘色的闪烁。在使用Theme.Holo的应用上上，会给你一个蓝色的闪烁。在使用Theme.Matrial的应用上，它会给你一个波浪动画。当然，你可以能提供自己的给selectableItemBackground的覆盖值从而换成你自己的StateListDrawable。

这个方案的精简就是视图比行其余部分内容在x轴线上要高。在此例中，因为行的剩余部分没有交互的，这就不是个问题。但是，如果我们把交互的控件放到行中，例如放一个CheckBox来实现一个清单－现在我们的视图会阻止用户跟这样控件交互。

####选项＃2:背景选择器

另一个应用selectableItemBackground到我们已存在的行内容的方法，而不是在行内容上加上一层。这是RecyclerView／CardRippleList2中所采用的方法。这里呢，selectableItemBackground被应用到了Cardview中的LineraLayout上。

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.v7.widget.CardView
 	 xmlns:android="http://schemas.android.com/apk/res/android"
  	xmlns:cardview="http://schemas.android.com/apk/res-auto"
  	android:layout_width="match_parent"
  	android:layout_height="wrap_content"
 	 android:layout_margin="4dp"
  	cardview:cardCornerRadius="4dp">

 	 <LinearLayout
    	android:layout_width="match_parent"
    	android:layout_height="wrap_content"
    	android:orientation="horizontal"
    	android:background="?android:attr/selectableItemBackground">

    <ImageView
      android:id="@+id/icon"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_vertical"
      android:padding="2dip"
      android:src="@drawable/ok"
      android:contentDescription="@string/icon"/>

    <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

      <TextView
        android:id="@+id/label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="25sp"
        android:textStyle="bold"/>

      <TextView
        android:id="@+id/size"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="15sp"/>
    </LinearLayout>

 	 </LinearLayout>

	</android.support.v7.widget.CardView>


对于没有交互的控件，像Textviews和ImageView，触摸事件会传到LinearLayout,它出发了作为LinearLayout背景的stateListDrawable状态的变化。并且，如果我们让行游交互控件，这些控件会仍可以处理它们自己的触摸事件，就像在这章之后内容看到的。

但是，特别是对这个样例应用而言，视觉效果很大程度上是和CardRippleList一样的，用户会得到基于所使用的活动主题中的selectableItemBackground的点击反馈。

####选择＃3 受控的波浪产生点

尽管如此两种点击实现都有一个问题，波动是从每行的中心开始的。

根据MaterialDesign规则，波浪应该从触摸事件产生的地方开始，所以呢它们能看到从指尖流出的感觉。

为了做这个，你需要使用setHotspot()方法，是在API Level 21的时候添加到Drawable中的。setHotspot给drawable提供了一个热点。并且RippleDrawable似乎使用了这个作为波浪效果的产生点。setHotspt（）取得一对浮点值，推测起来在OnTouchListener中着手使用了setHotspot（）,因为MotionEvent报告了接触点的x/y位置的浮点值。

RecyclerView/CardRippleList3样例复制了CardRipple3并添加了这个属性。

这个行布局是跟之前一样的。但是，在RowController中，当设置行的时候，我们注册了一个onTouchListener,为了找出当用户触摸我们行的时候的低级MotionEvent:

	class RowController extends RecyclerView.ViewHolder
   	 implements View.OnClickListener {
  	TextView label=null;
  	TextView size=null;
  	ImageView icon=null;
 	 String template=null;

 	 RowController(View row) {
    	super(row);

   	 label=(TextView)row.findViewById(R.id.label);
    	size=(TextView)row.findViewById(R.id.size);
   	 icon=(ImageView)row.findViewById(R.id.icon);

    	template=size.getContext().getString(R.string.size_template);

    	row.setOnClickListener(this);

   	 if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
     	 row.setOnTouchListener(new View.OnTouchListener() {
       	 @TargetApi(Build.VERSION_CODES.LOLLIPOP)
       	 @Override
       	 public boolean onTouch(View v, MotionEvent event) {
        	  v
            .findViewById(R.id.row_content)
            .getBackground()
            .setHotspot(event.getX(), event.getY());

          return(false);
      	  }
      	});
   	 }
 	 }

我们只在API等级21以上的上注册这个监听，因为在之前的Android版本上没有setHotSpot（）方法所以不需要监听。但是，如果我们在Android 5.0 以上的设备上，我们劫持触摸事件，把它传递给背景Drawable，并且返回false确保常规的触摸事件流程。

效果是细微的并且可能很难让你注意到。但是，如果你放慢看的话（例如，屏幕录制一段，然后逐帧逐帧地看），你会看到波浪效果是从接触点产生的，而不是像之前那样从中心处产生。并且，因为这个逻辑只在API等级21以上使用，较老的设备不受影响。


####游标呢？

目前来说，我们的模型数据一直是一个简单的静态数组。但是，时长我们需要使用从数据库或内容提供者收集的模型数据。可能由于其它原因，我们想把查询到的游标转化成常见java对象的数组。但是，没有什么能阻碍我们把游标作为RecyclerView的数据模型使用。

RecyclerView.Adapter是负责教导RecyclerView.ViewHolder模型数据绑定的。RecyclerView.Adapter基类对模型数据是如何组织的是茫然的，是数组,数组列表，游标还是json数组全然不知。并且实际上RecyclerView.ViewHolder的数据绑定逻辑，再次是我们的职责。基类对数据从哪里来是不在意的。因此，我们可以创建传递数据模型到RecyclerView需要位置的我们自己的协议。如果我们想使用Cursor作为手段，我们是被欢迎这么做的。

这在RecyclerView/VideoList这个样例项目中进行了阐释，它是在媒体内容提供者那章介绍项目的一个拷贝。在原先的样例中，列表是Listview；在这个样例中，列表是RecyclerView。

应用的核心“管道工程”是和之前的RecyclerView样例相似的，例如使用RecyclerViewActivity来处理在屏幕上获取RecyclerView。但是，现在我们的行布局是基于原先的视频列表行：

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:orientation="horizontal"
		android:padding="8dp"
 	 android:background="?android:attr/selectableItemBackground">

		<ImageView
			android:id="@+id/thumbnail"
			android:layout_width="64dp"
			android:layout_height="64dp"
			android:contentDescription="@string/thumbnail"/>

		<TextView
			android:id="@android:id/text1"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:layout_marginLeft="8dp"
			android:layout_gravity="center_vertical"
			android:textSize="24sp"/>

	</LinearLayout>


相似的，MainActivity的onCreate（）现在处理了Univeral Image Loader(UIL)设置（图片加载）并且剔除了来自MediaStore(initLoader())的loader。此外，onCreate()明确说明了给RecyclerView一个竖直滚动的LinearLayoutManager，并指出内容是来自一个
VideoAdapter:


	public class MainActivity extends RecyclerViewActivity implements
   	 LoaderManager.LoaderCallbacks<Cursor> {
  	private ImageLoader imageLoader;

 	 @Override
 	 public void onCreate(Bundle icicle) {
   	 super.onCreate(icicle);

   	 ImageLoaderConfiguration ilConfig=
        new ImageLoaderConfiguration.Builder(this).build();

   	 imageLoader=ImageLoader.getInstance();
   	 imageLoader.init(ilConfig);

    	getLoaderManager().initLoader(0, null, this);

    	setLayoutManager(new LinearLayoutManager(this));
    	setAdapter(new VideoAdapter());
  	}


CursorLoader逻辑，获取来自MediaStore的详细信息，除了当videoAdapter准备好时，给它提供Cursor之外，跟之前是差不多一样的：

 	 @Override
  	public Loader<Cursor> onCreateLoader(int arg0, Bundle arg1) {
   	 return(new CursorLoader(this,
                            MediaStore.Video.Media.EXTERNAL_CONTENT_URI,
                            null, null, null,
                            MediaStore.Video.Media.TITLE));
  	}

 	 @Override
 	 public void onLoadFinished(Loader<Cursor> loader, Cursor c) {
   	 ((VideoAdapter)getAdapter()).setVideos(c);
 	 }

  	@Override
  	public void onLoaderReset(Loader<Cursor> loader) {
    ((VideoAdapter)getAdapter()).setVideos(null);
 	 }

VideoAdapter是另一个RecyclerView.Adapter的子类，这次呢聪明地处理把游标作为模型数据的来源：
<pre>
<code>
class VideoAdapter extends RecyclerView.Adapter<RowController> {
    Cursor videos=null;

    @Override
    public RowController onCreateViewHolder(ViewGroup parent, int viewType) {
      return(new RowController(getLayoutInflater()
                                 .inflate(R.layout.row, parent, false),
                               imageLoader));
    }

    void setVideos(Cursor videos) {
      this.videos=videos;
      notifyDataSetChanged();
    }

    @Override
    public void onBindViewHolder(RowController holder, int position) {
      videos.moveToPosition(position);
      holder.bindModel(videos);
    }

    @Override
    public int getItemCount() {
      if (videos==null) {
        return(0);
      }

      return(videos.getCount());
    }
  }
</code>
</pre>

具体的：

* getItemCount（）返回来自cusor的视频个数，如果Cursor为空就是0（模仿CursorAdapter的行为，仅仅是把空的cursor当成没有行的cursor）
* onCreateViewHolder（）传递ImageLoader给RowController，那么所有的行共享一个共同的ImageLoader实例。
* onBindViewHolder（）移动游标到期望的位置，然后传递Cursor给RowController


同时注意到我们有一个setVideos（）方法用来把我的视频信息的游标和adapter联系起来。这也同时触发了notifyDataSetChanged()的调用，确保RecyclerView知道我们的模型已经改变了并且它应该重新渲染它的内容。

RowController构造器把ImageLoader存放进一个成员变量(imageLoader),此外从行中取得必须的控件然后设置点击事件。

  	RowController(View row, ImageLoader imageLoader) {
   	 	super(row);
    	this.imageLoader=imageLoader;

    	title=(TextView)row.findViewById(android.R.id.text1);
    	thumbnail=(ImageView)row.findViewById(R.id.thumbnail);

   	 	row.setOnClickListener(this);
  	}


在VideoAdapter上由onBindViewHolder（）动用的bindModel()方法使用和的是和原本的视频列表组装行布局的相同逻辑，此外在成员变量中持有当前行的Uri和MIME类型：

	void bindModel(Cursor row) {
   	 title.setText(row.getString(row.getColumnIndex(MediaStore.Video.Media.TITLE)));

   	 Uri video=
      	  ContentUris.withAppendedId(MediaStore.Video.Media.EXTERNAL_CONTENT_URI,
           	 row.getInt(row.getColumnIndex(MediaStore.Video.Media._ID)));
   	 DisplayImageOptions opts=new DisplayImageOptions.Builder()
        .showImageOnLoading(R.drawable.ic_media_video_poster)
        .build();

    	imageLoader.displayImage(video.toString(), thumbnail, opts);

    	int uriColumn=row.getColumnIndex(MediaStore.Video.Media.DATA);
    	int mimeTypeColumn=
       	 row.getColumnIndex(MediaStore.Video.Media.MIME_TYPE);

    	videoUri=row.getString(uriColumn);
    	videoMimeType=row.getString(mimeTypeColumn);
  	}


onClick()方法使用这些保存的Uri和MIME类型值来启动播放所选择的视频的活动：


	 @Override
 	 public void onClick(View v) {
   	 Uri video=Uri.fromFile(new File(videoUri));
   	 Intent i=new Intent(Intent.ACTION_VIEW);

    	i.setDataAndType(video, videoMimeType);
    	title.getContext().startActivity(i);
  	}


除了缺少分割线，用户界面和原本的视频列表很相似。

####网格

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

在我们的例子中，我们请求了两个跨度，结果就变成了两列：

![wizard](/the_busy_coder's_guide/img/figure_437.png)

在此例中，有一个真的有行有列的单元格组成的网格。因此，行的高度是有行中最高的单元格决定的。举个例子，“amet单元格”比它所需的高是因为同一行的“consectetuer”单元格。在这章的之后内容中，我们会检视另外一个选项，StaggeredGridLayoutManager，在里面单元格不必要整齐的排列。

####选择列的数量

如果我们旋转这个样例的屏幕，你会决定看起来好一点，因为它们是改变用途为列表样式行了：

![wizard](/the_busy_coder's_guide/img/figure_438.png)

但是，一些应用可能有很小的单元。此外，我们还要考虑平板，甚至可能电视。你可能需要给予屏幕尺寸和方向来决定垮度。

一种尝试是使用整型资源。你可以有一个有<integer>元素的res/values/ints.xml文件，给出整型的名称和值。你也可以有res/values-w600dp或其它资源变化，这样你就能给不同的屏幕尺寸提供不同的值。然后，在运行时，调用getResources().getInteger（）来获取给当前设置使用的资源的正确值，并在你的GridLayoutManager构造器中使用。现在你能通过提供给构造函数的跨度来控制存在多少列了。

另一种尝试是，Chiu-ki Chan提议的，创建一个RecyclerView的一个子类，在其中你提供了一个自定义属性给大只的列宽度。然后在你的子类的onMeasure()方法中，你可以计算出跨度的数量从而得出你想要的列宽度。

当然，另一种利用屏幕控件的方式是增大单元格。默认情况下，它们会均匀增大，因为每个单元格占据一个跨度，并且跨度是均匀大小的。但是，你可以通过附加一个GridLayoutManager.SpanSizeLookup到GridLayoutManager改变这个行为.GridLayoutManager是用于负责指示给位置的项应该占据网格的多少跨度的。我们会在这章之后内容进行调查它是怎么工作的。

###改变项
目前为止，所有RecyclerView中的项有着相同的基础结构，只是这些项中控件的内容不同。但是，我们需要基于不同布局的有着实质不同的项也是完全有可能的。ListView及其亲属通过ListAdapter中的getViewTypeCount()和getItemViewType（）对其进行处理。RecyclerView和RecyclerView.Adapter提供了一个相似的机制，包括它们自己的getItemViewType（）变种方法。在这节中，我们会检视这个是如何在列表和网格。

####有头的列表

有很多情况下，我们必须有某种类型的头。这些头的样子通常和其余行有着很大的区别，因此处理这个问题的最好方法是教导适配器关于多种行类型的事。

这个能在RecyclerView/HeaderList这样样例项目中看到。这是一个相似ListView项目的一个拷贝，其中我们会把25个latin单词分成5个一组，每组有着它自己的头。

因此，现在我的数据模型是一个二维字符串数组:

 	private static final String[][] items= {
      { "lorem", "ipsum", "dolor", "sit", "amet" },
      { "consectetuer", "adipiscing", "elit", "morbi", "vel" },
      { "ligula", "vitae", "arcu", "aliquet", "mollis" },
      { "etiam", "vel", "erat", "placerat", "ante" },
      { "porttitor", "sodales", "pellentesque", "augue", "purus" } };


现在我们的getItemCount()方法需要把头和常规的行算在内。每一组都有一个头，所以getItemCount()统计了有着额外标题的组的数量:

	@Override
   	 public int getItemCount() {
      	int count=0;

      	for (String[] batch : items) {
        	count+=1 + batch.length;
      	}
     	 return(count);
    	}

为了教导RecyclerView不同的行，我们需要实现getItemViewType().不像其对应的ListAdapter,getItemViewType（）可能返回任意的int值，只要在行类型中是唯一的。实际上，推荐的方式是使用专有的ID资源来确保唯一性。

为此，我们在res/values/id.xml文件中定义了两个id资源。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
  	<item type="id" name="header"/>
  	<item type="id" name="detail"/>
	</resources>


然后，getItemViewType就能返回R.id.header或R.id.detail来唯一识别两种行类型并且明确那种类型对应哪个位置：

 	@Override
    public int getItemViewType(int position) {
      if (getItem(position) instanceof Integer) {
        return(R.id.header);
      }

      return(R.id.detail);
    }

    private Object getItem(int position) {
      int offset=position;
      int batchIndex=0;

      for (String[] batch : items) {
        if (offset == 0) {
          return(Integer.valueOf(batchIndex));
        }

        offset--;

        if (offset < batch.length) {
          return(batch[offset]);
        }

        offset-=batch.length;
        batchIndex++;
      }

      throw new IllegalArgumentException("Invalid position: "
          + String.valueOf(position));
    }


这里借助了这样样例的最初的ListView版本的一个拷贝，它返回了header项的int值以及一个详细项的字符串。注意getItem不是RecyclerView.Adapter协议的一部分，但是你也是可以这么做的。

在onCreateViewHolder()中，我们现在可以开始关注第二个参数，之前一直被我们忽略。viewType这个值，是从getviewType返回的值，并且它指示了我们应该返回何种类型的
RecyclerView.ViewHolder。在我们的情形中，有着两种可能性，所以呢我们指示inflat了合适的布局并使用一个专有的控制类(用于headers的HeaderController和用于detail的RowController):


	@Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
      if (viewType==R.id.detail) {
        return(new RowController(getLayoutInflater()
                     .inflate(R.layout.row, parent, false)));
      }

      return(new HeaderController(getLayoutInflater()
                    .inflate(R.layout.header, parent, false)));
    }


相似的，我们在onBindViewHolder（）中的绑定逻辑需要找到正确类型的模型信息对应合适的controller：

     @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
      if (holder instanceof RowController) {
        ((RowController)holder).bindModel((String)getItem(position));
      }
      else {
        ((HeaderController)holder).bindModel((Integer)getItem(position));
      }
    }


RowController是跟之前样例一样的设置步骤。HeaderController也一样，尽管它简单很多，因为我们只需要更新一个控件，而且不需要关心点击事件。

    package com.commonswares.android.recyclerview.headerlist;
    
    import android.support.v7.widget.RecyclerView;
    import android.view.View;
    import android.widget.TextView;

  	class HeaderController extends RecyclerView.ViewHolder{
      TextView label=null;
      String template=null;

      HeaderController(View row){
        super(row);
        
        label=(TextView)row.findViewById(R.id.label);

        template=label.getContext().getString(R.string.header_template);

      }

      void bindModel(Integer headerIndex){
        label.setText(String.format(template,headerIndex.intValue()+1));
      }
  }



结果是头行是一种感觉，详情行又是另一种感觉：

![wizard](/the_busy_coder's_guide/img/figure_439.png)

####网格形势的表格

在RecyclerView网格的讨论中，我们看到了一种利用大屏而有更多单元的方式，部分通过在屏幕上有更多跨度实现。

另一种利用屏幕空间的方式是增大单元格。默认情况下，它们会平均增长，因为每个单元格都占据一个跨度，并且跨度是一样大小的。
但是，你可以通过附加一个GridLayoutManager.SpanSizeLookup来改变这个行为方式。GridLayoutManager.SpanSizeLookup是负责指示给定项的位置的，以及它应该在网格中占据多少跨度。

一种应用GridLayoutManager.SpanSizeLookup的方式就是做一张表格。如果你需要一张表格，但是用户只能选择行的话，
那就是使用LinearLayoutManager并在在每行中使用连续的单元格的问题。例如，每行可以是水平的LinearLayout并且列宽是
使用android:layout_weight决定的。但是有时候，你想要一张这么样的表格。表格中的独立单元格可以点击（或者像轨迹球一样通过五个方向的进行选择）。
在此例中，GridLayoutManager.SpanSizeLookup会让你指示，你输出的列中的单元应该占据多少跨度。通过对没列使用始终如一的跨度
，你就能得到相同权重的列宽，而你可能使用基于LinearLayoutManager支持的Recyclerview中LinearLayout也能得到。

当你看到一个例子的时候会有更多的感觉。

RecyclerView/VideoTable样例项目是从VideoList样例项目拷贝过来的，并修改了一些地方：

* 我们要使用GridLayoutManager，依然把我的的输出弄进逻辑上的行，每行有着三个单元(标题，缩略图，视频持续时间)
* 我们要使用GridLayoutManager.SpanSizeLookup来控制网格中每列的宽度
* 因为我们的单元有着多样的内容(一个ImageView以及其他的TextView),对于这些单元我们会使用不同的Controller，针对单元的内容类型进行优化。

含有文本（标题和播放时长)这两列会使用以下的布局：

	<LinearLayout xmlns:android:"http://schemas.android.com/apk/res/android"
 	 android:layout_width="match_partent"
  	android:layout_height="wrap_content"
  	android:orientation="horizontal"
 	 android:padding="8dp"
 	 android:background="?android:attr/selectedItemBackground">
 	 <TextView 
   	 android:id="@android:id/text1"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:layout_marginleft="8dp"
     android:layout_gravity="center_vertical"
     android:textSize="24sp"/>
   </LinearLayout>


LinearLayout根元素看起来是多余的，但是我们把它用作selectableitemBackground，在单元被点击的时候提供一个反馈。

相似的，我们又一个缩略图专有的布局：

	<LinearLayout xmlns:android:"http://schemas.android.com/apk/res/android"
 	 android:layout_width="match_partent"
 	 android:layout_height="wrap_content"
 	 android:orientation="horizontal"
  	android:padding="8dp"
  	android:background="?android:attr/selectedItemBackground">
  	<ImageView 
   	 android:id="@android:id/text1"
     android:layout_width="96dp"
     android:layout_height="72dp"
     android:layout_gravity="center_vertical"
     android:textSize="24sp"
     android:contentDescription="@string/thumbnail"
     />
   å</LinearLayout>

MainActivity中的onCreate()的绝大部分还是跟之前一样的。尽管如此，我们创建了一个ColumnWeightSpanSizeLookup类的实例用作两件事情

1.调用它的getTotalSpans()来告诉GridLayout使用多少跨度

2.把它作为一个GridLayoutManager.SpanSizeLookup通过setSpanSizeLookup()来附加到GridLayoutManager:


	@Override
	public void onCreate(Bundle icicle){
	super.onCreate(icicle);

	ImageLoaderConfiguration icConfig=new ImageLoaderConfiguration.Builder(this).build();

	imageLoader=ImageLoader.getInstance;
	imageLoader.init(ilConfig);

	getLoaderManager().initLoader(0,null,this);

	ColumnWeightSpanSizeLookup spanSizer=new ColumnWeightSpanSizeLookup(COLUMN_WEIGHTS);
	GridLayoutManager mgr=new GridLayoutManager(this,spanSizer.getTotalSpans());

	mgr.setSpanSizeLookup(spanSizer);
	setLayoutManager(mgr);
	setAdapter(new VideoAdapter());
	}


后一点意味着ColumnWeightSpanSizeLookup 是抽象的GridLayoutManager.SpanSizeLookup基类的子类。你需要在GridLayoutManager.SpanSizeLookup()中重载的方法是
getSpanSize().给定一个项的位置，getSpanSize()就能返回这个单元项应该占据的跨度。

（在Android中我们会提到跨度很多次）

ColumnWeightSpanSizeLookup通过一系列的列权重来进行处理，结果它在constructor.onCreate（）中得到了一个引用COLUMN_WEIGHTS权重静态数据的int类型数组：

	private static final int[] COLUMN_WEIGHTS={1,4,1};

这个int数组告诉我们有多少列以及用每个列的跨度。


把位置转换成列的序号是一个实现取模操作的过程，所以在ColumnWeightSpanSizeLookup中getSpanSize()的实现仅仅返回目标列的列权重

	package com.commonsware.android.recyclerview.videotable;
 	import android.support.v7.widget.GridLayoutManager;

 	class ColumnWeightSpanSizeLookup extends GridLayoutManager.SpanSizeLookup{
 	 private final int[] columnWeights;

     ColumnWeightSpanSizeLookup(int[] columnWeights){
     	this.columnWeights=columnWeights;
     }

     @Override
     public int getSpanSize(int position){
     	return(columnWeights[position%columnWeights.length]);
     }

     int getTotalSpans(){
     	int sum=0;

     	for(int weight:columnWeights){
     		sum+=weight;
     	}

     	return(sum);
     }
	}

getTotalSpans是一个方便的方法，它会统计列权重的总和。也就是GridLayoutManager总共会使用跨度，每个列根据int数组中的值来获取特定的跨度值。
此例中，虽然注意当我们对int数组硬编码，但我们也可以使用<integer-array>把这些值从Java代码中抽离出来，并且可能屏幕尺寸或其它配置的变化而变化。

所有这些会帮我们设置好输出的网格的跨度和列的跨度。两者结合会给我们一个行结构，因为每行的列宽使用了行的所有跨度，强制GridLayoutManager把随后的项
放在下一行。

项目的其余部分则关注于使用getItemViewType（）等来让这些不同的单元有不同的布局。

VideoAdapter实现getItemViewType（）仅仅返回了位置对3进行取模操作的结果（在此例中，是0，1，2）：

  @Override
  public int getItemViewType(int position){
     return(position%3);
  }

getItemCount()按每个视频三个个单元计数，所以项数是是视频数的三倍：

  @Override
  public int getItemCount(){
    if(video==null){
      return (0);
    }
     return(videos.getCount()*3);
  }

onCreateViewHolder()和onBindViewHolder()方法吧三个项类型计算在内，根据项的类型而使用VideoThumbnailController或VideoTextController。这些
类都是从一个BaseVideoController继承而来的，它定义了onBindViewHolder（）能使用的bindModel（）方法。

  @Override
  public BaseVideoController onCreateViewHolder(View parent,int viewType){
     BaseVideoController result=null;

     switch(viewType){
      case 0:
         result=new VideoThembnailController(getLayoutInfalter().inflate(R.layout.thumbnail,parent,false),imageLoader);
         break;
      case 1:
         int cursorColumn=video.getColumnIndex(MediaStore.Video.VideoColumns.DISPLAY_NAME); 
         result=new VideoTextController(getLayoutInflater().inflate(R.layout.label,parent,false),
           android.R.id.text1,cursorColumn
         );
         break;

      case 2:
          cursorColumn=videos.getColumnIndex(MediaStore.Video.VideoColumns.DURATION);
          result=new VideoTextController(getLayoutInfalter().infalte(R.layout.lable,parent,false),android.R.id.text1,cursorColumn);
       break;   
     }
     return(result);
 	 } 

	  void setVideos(Cursor videos){
     	this.videos=videos;
     	notifyDataSetChanged();
 	 	}

 	 @Override
 	 public void onBindViewHolder(BaseVideoController holder,int position){
    	videos.moveToPosition(position/3);
    	holder.bind(videos);
 	 }

BaseVideoController处理单元格的点击事件以及点击事件用到的视频的Uri和MIME类型。

 	 package com.commonsware.android.recyclerview.videotable;

  	import android.content.Intent;
  	import android.database.Cursor;
  	import android.net.Uri;
  	import android.provider.MediaStore;
  	import android.support.v7.widget.Recyclerview;
  	import android.view.View;
  	import com.nostra13.universalimageloader.core.ImageLoader;
  	import java.io.File;

  	abstract class BaseVideoController extends RecyclerView.ViewHolder
      implements View.onClickListener{
        private String videoUri=null;
        private String videoMimeType=null;

        BaseVideoController(View cell){
          super(cell);

          cell.setonClickListener(this);
        }

        @Override
        public void onClick(View v){
          Uri video=Uri.fromFile(new File(videoUri));
          Intent i=new Intent(Intent.ACTION_VIEW);

          i.setDataAndType(video,videoMimeType);
          itemView.getContext().startActivity(i);
        }

        void bindModel(Cursor row){
          int uriColumn=row.getColumnIndex(MediaStore.Video.Media.DATA);
          int mimeTypeColumnIndex(MediaStore.Video.Media.MIME_TYPE);

          videoUri=row.getString(uriColumn);
          videoMimeType=row.getString(mimeTypeColumn);
        }
     }

VideoTextController 扩展了BaseVideoController并把来自MediaStoreCursor的列和TextView进行id绑定：

  package com.commonsware.android.recyclerview.videotable;

  import android.database.Cursor;
  import android.view.View;
  import android.widget.TextView;

  class VideoTextController extends BaseVideoController{
    private TextView label=null;
    private int cursorColumn;

    VideoTextController(View cell,int lableId,int cursorColumn){
      super(cell);
      this.cursorColumn=cursorColumn;

      label=(TextView)cell.findViewById(labelId);
    }

    @Override
    void bindModel(Cursor row){
      super.bindModel(row);

      label.setText(row.getString(cursorColumn));
    }

  	 }

VideoThumbnailController使用UIL异步获取视频缩略图并把它绑定到单元视图的ImageView中：

   	 package com.commonsware.android.recyclerview.videotable;

   	 import android.content.ContentUris;
     import android.database.Cursor;
     import android.net.Uri;
     import android.provider.MediaStore;
     import android.view.View;
     import android.widget.Imageview;
     import com.nostra13.universalimageloader.core.DispalyImageOptions;
     import com.nostra13.universalimageloader.core.ImageLoader;

     class VideoThumbnailController extends BaseVideoController{
      super(cell);
      this.imageLoader=imageLoader;

      thumbnail=(ImageView)cell.findViewById(R.id.thumbnail);
    }

    @Override
    void bindModel(Cursor row){
      super.bindModel(row);

      Uri video=ContentUris.withAppendedId(MediaStore.Video.Media.EXTERAL_CONTENT_URI,
        row.getInt(row.getColumnIndex(MediaStore.Video.Media._ID)));
        DisplayImageOptions opts=new DisplayImageOptions.Builder()
        .showImageOnLoading(R.drawable.ic_media_video_poster)
        .build();

        imageLoader.dispalyImage(video.toString,thumbnail,thumbnail,opts);
        )
    }

结果是和原本的VideoListdemo相同的信息，但是组织成table的形式，每个单元都是可以的点击的：

![wizard](/the_busy_coder's_guide/img/figure_440.png)

持续时间是由MediaStore以毫秒形式返回的，直接展示给用户不是很好的选择。改进版的这个应用可能使用一个专有的RecyclerView.
ViewHolder会把毫秒数转换成时,分，秒。（例如，以HH:MM:SS的形式展示给用户）

也要注意到单元格的大小是完全受它们的权重影响，它不可能把所有内容配置得很好。选择的权重在10寸的平板上就差强人意：

![wizard](/the_busy_coder's_guide/img/figure_441.png)

可变的行内容

目前为止，我们所使用的所有的项都只是展示。最多，它们可能反馈下点击事件，和点击一个Listview和GridView的单元一样。

但是，选择模式是怎么样的呢？

ListView和GridView－以及它们共同的祖先AbsListView-有着选择模式的概念，用户可以选中或取消选中项，并且它们的列表或网格会保存这些状态。

因为有很多事情到涉及在RecyclerView中，RecyclerView并不提供选择模式...尽管如此，你可以自己实现它。RecyclerView/ChoiceList
样例代码就把我们列表样式的RecyclerView转化成一个核对列表，每一行都有一个CheckBox,RecyclerView.Adapter会为我们保存CheckBox的选中状态。

首先，我们需要在行中添加一个CheckBox：

  <?xml version="1.0" encoding="utf-8?"
  <android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:cardview="http//shcemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="4dp"
    cardview:cardCornerRadius＝"4dp">
    <LinearLayout
      android:id="@+id/row_content"
      android:layout_width="match_partent"
      android:layout_height="wrap_content"
      android:orientation="horizontal"
      android:background="?android:attr/selectableItemBackground">
      <ImageView
       android:id="@+id/icon"
       android:layout_width="wrap_parent"
       android:layout_height="wrap_content"
       android:padding="2dip"
       android:src="@drawable/ok"
       android:contentDescription="@string/icon"/>
       <LinearLayout
         android:layout_width="0dp"
         android:layout_height="wrap_content"
         android:layout_weight="1"
         android:orientation="vertical">
         <TextView
          android:id="@+id/label"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:textSize="25sp"
          android:textStyle="bold"/>
          <TextView 
           android:id="@+id/size"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:textSize="15sp"/>
        </LinearLayout>
        <CheckBox
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:id="@+id/cb"
          android:layout_gravity="center_vertical"/>
        </LinearLayout>   
  </android.support.v7.widget.CardView>
    
我们的IconicAdapter 跟之前的只有细微的不同：

* 它从一个ChoiceCapableAdapter继承而来，我们稍后机会过以下，并且
* 它提供给ChoiceCapableAdapter一个MultiChoiceMode实例作为链接到ChoiceCapableAdapter构造器的一部分

   class IconicAdapter extends ChoiceCapableAdapter<RowController>{
    IconicAdapter(){
       super(new MultiChoiceMode());
    }
    
    @Override
    public RowController onCreateViewHolder(ViewGroup parent,int viewType){
      return(new RowController(this,getLayoutInfalter().infalte(R.layout.row,parent,false)));
    }

    @Override
    public void onBindViewHolder(RowController holder,int position){
       holder.bindModel(items[positoin]);
    }

    @Override
    public int getItemCount(){
       return(items.length);
    }

   }

ChoiceCapableAdapter只不过是一个知道如何处理选择模式的RecyclerView.Adapter，以实现ChocedMode接口的方式:

   	package com.commonsware.android.recyclerview.chocelist;

  	 import android.os.Bundle;

  	 public interface ChoiceMode{
      void setChecked(int position,boolean isChecked);
      boolean isChecked(int position);
      void onSaveInstanceState(Bundle state);
      void onRestoreInstanceState(Bundle state);
 	  }  

  ChoiceMode 是一个有效的策略类，负责记录选择状态，不只是当前的ChoiceCapableAdapter实例，还包括将来产生作为配置改变一部分的。它需要四个方法：

  * setChecked()和isChecked()是给定位置是否选中的getter和setter。
  * onSaveInstanceState()和onRestoreInstanceState()管理存储和恢复这些来自活动或者碎片的保存的实例状态Bundle.

这个项目使用了ChoiceMode的多选实现：
  
  	 package com.commonsware.android.recyclerview.choicelist;

  	 import android.os.Bundle;

 	 public class MultiChoiceMode implements ChoiceMode{
     private static final String STATE_CHECK_STATES="checkstates";
     private ParcelabelSparseBooleanArray checkStates=new ParcelableSparseBooleanArray();

     @Override
     public void setChecked(int position,boolean isChecked){
       checkStates.put(position,isChecked);
     }

     @Override
     public boolean isChecked(int position){
      return(checkStates.get(position,false));
     }

     @Override
     public void onSaveInstanceState(Bundle state){
       state.putParcelable(STATE_CHECK_STATES,checkStates);
     }

     @Override
     public void onRestoreInstanceState(Bundle state){
      checkStates=state.getParcelabel(STATE_CHECK_STATES);
     }
  }

MultiChoiceMode,反过来，大部分是通过一个ParcelabelSparseBooleanArray进行处理的。SparseBooleanArray是一个由AndroidSDK提供的类，是一个在空间方面十分高效对
int值到boolean值的映射，对应着使用HashMap把这些基本类型转换为Integer和Boolean对象。但是，由于令人费解的原因，SparseBooleanArray没有实现Parcelable,因此它不能
存在Bundle中。ParcelableSparseBooleanArray是SparseBooleanArray的子类，它可以处理Parcelable方面的东西：

 	 package com.commonsware.android.recyclerview.choicelist;

  	 import android.os.Parcel;
  	 import android.os.Parcelable;
  	 import android.util.SparseBooleanArray;

 	 public class ParcelableSparseBooleanArray extends SparseBooleanArray
       implements Parcelable{
        public static Parcelabel.Creator<ParcelableSparseBooleanArray> CREATOR
        =new Parcelable.Creator<ParcelableSparseBooleanArray>(){
          @Override
          public ParcelableSparseBooleanArray createFromParcel(Parcel source){
            return(new ParcelableSparseBooleanArray(source));
          }
          @Override
          public ParcelableSparseBooleanArray[] newArray(int size){
            return(new ParcelbleSparseBooleanArray[size]);
          }
        };

        public ParcelableSpaseBooleanArray(){
          super();
        }

        private ParcelableSparseBooleanArray(Parcel source){
          int size=source.readInt();

          for(int i=0;i<size;i++){
            put(source.readInt(),(Boolean)source.readValue(null));
          }
        }

        @Override
        public int describeContents(){
          return(0);
        }

        @Override
        public void writeToParcel(Parcel dest,int falgs){
          dest.writeInt(size());

          for(int i=0;i<size();i++){
            dest.writeInt(keyAt(i));
            dest.writeValue(valueAt(i));
          }
        }
        
       }

实际结果是多选模式可以借助ParcelableSparseBooleanArray,记录特定位置的选择状态。

ChoiceCapaleAdapter，然后就是一个可以显现选择模式实现的RecyclerView.ViewHolder:
  
 	 package com.commonsware.android.recyclerview.choicelist;

 	 import android.os.Bundle;
 	 import android.support.v7.widget.RecyclerView;

 	 abstract public class
        ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
        extends RecyclerView.Adapter<T>{
          private final ChoiceMode choiceMode;

          public ChoiceCapableAdapter(ChoiceMode choice){
            super();
            this.choice=choiceMode;
          }

          void onChecked(int position,boolean isChecked){
            choiceMode.setChecked(position,isChecked);
          }

          boolean isChecked(int position){
            return (choiceMode.isChecked(position));
          }

          void onSaveInstanceState(Bundle state){
            choiceMode.onSaveInstanceState(state);
          }

          void onRestoreInstanceState(Bundle state){
            choiceMode.onRestoreInstance(state);
          }
        }

由ChoiceCapableAdapter暴露出来的方法可以在外部使用。具体的，主活动把onSaveInstance()和onRestoreInstanceState()委托给ChoiceCapableAdapter。所以选中
状态可以掌握配置改变等等。此外，RowController可以勾住OncheckedChangedListener并更新基于checkbox状态改变的ChoiceCapableAdapter:

  	package com.commonsware.android.recyclerview.choicelist;

 	 import android.annotation.TargetApi;
 	 import android.os.Build;
 	 import android.support.v7.widget.RecyclerView;
 	 import android.view.MotionEvent;
  	 import android.view.View;
  	 import android.widget.CheckBox;
  	 import android.widget.CompoundButton;
  	 import android.widget.ImageView;
  	 import android.widget.TextView;
  	 import android.widget.Toast;

 	 class RowController extends RecyclerView.ViewHolder
       implements View.onClickListener,CompoundButton.OnCheckedChangeListener{
        private ChoiceCapableAdapter adapter;
        private TextView label=null;
        private TextView size=null;
        private ImageView icon=null;
        private String template=null;
        private CheckBox cb=null;

        RowController(ChoiceCapableAdapter adapter,view row){
          super(row);

          this.adapter=adapter;
          label=(TextView)row.findViewbyId(R.id.label);
          size=(TextView)row.findViewById(R.id.size);
          icon=(ImageView)row.findViewById(R.id.icon);
          
          template=size.getContext().getString(R.string.size_template);

          row.setOnclickListener(this);

          if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.LOLLOPOP){
            row.setOnTouchListener(new View.onTouchListener){
              @TargetApi(Build.VERSION_CODES_LOLLIPOP)
              @Override
              public boolean onTouch(View v,MotionEvent event){
                v
                 .findViewById(R.id.row_content);
                 .getBackgound()
                 .setHotspot(event.getX(),event.getY());

                 return(false);
              }
            });
          }

          cb.setOnCheckedChangeListener(this);
        }

        @Override
        public void onClick(View v){
          Toast.makeText(v.getContext(),String.format("clicked on position %d",getAdapterPosition()),
          Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onCheckedChanged(CompoundButton buttonView,boolean isChecked){
          adapter.onChecked(String.format(template,item.length()));
        }


        void bindModel(String item){
          label.setText(item);
          size.setText(String.format(template,item.length()));

          if(item.length()>4){
            icon.setImageResource(R.drawable.delete);
          }
          else{
            icon.setImageResource(R.drawable.ok);
          }

          cb.setChecked(adapter.isChecked(getAdapterPosition()));
        }
       }

这里，bindModel（）是基于ChoiceCapableAdapter给Recyclerview.ViewHolder的某一位置的isChecked()的值来更新CheckBox的（通过getPositon（）获取）。
onCheckedChanged()更新了ChoiceCapableAdapter来记录这个位置是否被选中，来处理行回收，配置改变等情况。

用户可以点击checkbox的时候checkbox会改变状态，这个状态在行回收，配置改变的时候都能存在。除此之外，用户界面没有什么变化。

![wizard](/the_busy_coder's_guide/img/figure_442.png)

注意，这个例子使用的Android5.0以上设备的Material主题，因为截屏是来自一个Android5.0的模拟器，CheckBox的样式是基于主色调的，这里的主色调是明黄色。

####转移到激活样式

也需要注意到ChoiceCapableAdpter,MultiChoiceMode及其亲属对用户是如何被通知什么选中了什么没被选中是茫然的。前一个例子中的RowController恰好使用了一个CheckBox。
RowController也可以使用其它空间，比如说Switch。

另一种方式就是使用激活状态。再一次的，这种事情也是由ListView及其选择模式自动处理的，但是只要一点调整，我们就能让我们的RowController使用这个方法。这会在RecyclerView／activatedList这个样例项目中展示。

首先，我们需要给我们的行的背景有一个支持激活状态的StateListDrawble。最简单的方法－也是Listview使用的传统方法－用固有的主题背景drawable设置一个激活样式，然后把样式应用到行上。

所以呢，这个样例应用就在res/values/styles.xml定义了activated：

	<?xml version="1.0" encoding="utf-8"
	<resource>

		<style name="Theme.AppTheme parent="@android:style/Theme.Holo.Light.DarkActionBar">
		</style>

		<style name="activated" parent="Theme.Apptheme">
		   <item name="android:background">?android:attr/activatedBackgroundIndicator</item>
		</style>   

	</resource>

注意，activated是从Theme.AppTheme继承而来的。这就意味着通常会得到Theme.Holo风格的背景，但是在API等级21以上，我们得到了Theme.Material风格的背景，由覆盖Theme.Apptheme的一个res/value-v21／styles.xml文件提供：

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
		<style name="Theme.AppTheme" parent="android:Theme.Material.Light.DarkActionBar"
		<item name="android:colorPrimary">@color/primary</item>
		<item name="android:colorPrimaryDark">@color/primary_dark</item>
		<item name="android:colorAccent">@color/accent</item>
	    </style>
	<resources>    			

我们的行布局现在弃用了CardView（它的背景可能和激活样式背景冲突）并应用激活样式到根LinearLayout上：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
	 xmlns:android:"http://shcemas.android.com/apk/res/android"
	 android:layout_width="match_parent"
	 android:layout_height="wrap_content"
	 android:orientation="horizontal"
	 style="@style/activated">
     
     <ImageView
      android:id="@+id/icon"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_vertical"
      android:padding="2dip"
      android:src="@drawable/ok"
      android:contentDescription="@String/icon"/>
    
    	<LinearLayout
     	 android:layout_width="0dp"
     	 android:layout_height="wrap_content"
      	android:layout_weight="1"
      	android:orientation="vertical">

     	 <TextView
      	 android:id="@+id/label"
      	 android:layout_width="wrap_content"
      	 android:layout_height="wrap_content"
      	 android:textSize="25sp"
       	 andorid:textStyle="bold"/>
    
      	<TextView
       	 android:id="@+id/size"
       	 android:layout_width="wrap_content"
       	 android:layout_height="wrap_content"
      	 android:textSize="15sp"/>
	 	</LinearLayout>
	 </LinearLayout>

这个行也没有CheckBox了，因为它不再需要。

RowController 现在使用OnClicklistener接口来响应点击并使用它来切换行的激活状态：

	@Override
	public void onClick(View v){
		boolean isCheckedNow=adapter.isChecked(GetAdapterPosition());

		adapter.onChecked(getAdapterPosition,!isCheckedNow);
		row.setActivated(!isCheckedNow);
	}

setActivated(),应用到视图，指示它有没有激活，影响所有依赖于状态的所有东西，例如背景。

类似的，bindModel()使用setActivated()更新绑定数据时的激活状态：

	void bindModel(String item){
		label.setText(item);
		size.setText(String.format(template,item.length()));

		if(item.length()>4){
           icon.setImageResource(R.drawable.delete);
		}
		else{
			icon.setImageResource(R.drawable.ok);
		}

		row.setActivated(adapter.isChecked(getAdapterPosition()));
	}


其余都跟最初的checkbox样例一样。只不过，选中状态用激活高亮指示。

![wizard](/the_busy_coder's_guide/img/figure_445.png)

并且,因为这个demo是在Android 5.o上运行的，激活的高亮色时主色调，而这里是被设置成黄色的。

####但是，单选呢？

两个之前的例子阐述的都是多选的行为。尽管如此，单选行为是更好的选择。例如,在主从结构中，在双窗格模式中(例如，平板，主页和详细页都是可见的)，你可能就要一个
单选模式了。

这肯定是可能的，尽管如此，再一次的RecyclerView没有提供。它也带来了个小问题，在选中其它项的时候，如何反选之前选中的项。就像RadioGroup中的RadioButton控件一样，
我们需要确保同一时间只有一个选中的，这就需要我们更新之前选中的和现在没选中的项的用户界面。

更改一些地方之后，我们使用多选激活状态列表的样例项目，可以改成单选。这些修改会在RecyclerView/SingleActivatedList样例项目中阐释。

ChoiceMode接口现在有了两个新的方法：

* 1.isSingleChoice()在单选模式的时候返回true，其余返回false
* 2.getCheckedPosition()会返回当前选中项的位置

		package com.commonsware.android.recyclerview.singleactivatedlist;
		import android.os.Bundle;

		public interface ChoiceMode{
			boolean isSingleChoice();
			int getCheckedPosition();
			void setChecked(int position,boolean isChecked);
			boolean isChecked(int position);
			void onSaveInstanceState(Bundle state);
			void onRestoreInstanceState(Bundle state);
		}


SingleChoiceMode 是现在我们的ChoiceMode的实现:

		package com.commonsware.android.recyclerview.singleactivatedlist;

		import android.os.Bundle;

		public class SingleChoiceMode implements ChoiceMode{
			private static final String STATE_CHECKED="checkedPosition";
			private int checkedPosition=-1;

			@Override
			public boolean isSingleChoice(){
				return(true);
			}

			@Override
			public int getCheckedPosition(){
				return(checkedPosition);
			}

			@Override
			public void setChecked(int position,boolean isChecked){
				if(isChecked){
					checkedPosition=position;
				}
				else if(isChecked(position)){
					checkedPosition=-1;
				}
			}

            @Override
            public boolean isChecked(int position){
            	return(checkedPosition==position);
            }
           
			@Override 
			public void onSaveInstanceState(Bundle state){
				state.putInt(STATE_CHECKED,checkedPosition);
			}

			@Override
			public void onRestoreInstanceState(Bundle state){
				checkPosition=state.getInt(STATE_CHECKED,-1);
			}
		}

单选模式记录了当前选中的位置，用－1指示没有位置选中。需要注意的是，如果一个位置被选中了，使用setChecked()进行反选，SingleChoiceMode就会返回－1指示当前没有选中。

ChoiceCapableAdapter也有两处修改，首先它接受RecyclerView本身作为构造器参数，以一个rv成员变量就像持有。并且，onChecked()需要进行修改负责在新的项选中时移除之前选中的激活状态：

		package com.commonsware.android.recyclerview.singleactivatedlist;

		import android.os.Bundle;
		import android.support.v7.widget.RecyclerView;

		abstract public class ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
		      extends RecyclerView.Adapter<T>{
		      	private final ChoiceMode choiceMode;
		      	private final RecyclerView rv;

		      	public ChoiceCapableAdapter(ChoiceMode choiceMode,ChoiceMode choiceMode){
		      		super();
		      		this.rv=rv;
		      		this.choiceMode=choiceMode();
		      	}

		      	void onChecked(int position,boolean isChecked){
		      		if(choiceMode.isSingleChoice()){
		      			int checked=choiceMode.getCheckedPosition();

		      			if(checked>=0){
		      				RowController row=(RowController)rv.findViewHolderForAdapterPosition(checked);
		      				if(row!=null){
		      					row.setChecked(false);
		      				}
		      			}
		      		}
		      	}

		      	boolean isChecked(int position){
		      		return(ChoiceMode.isChecked(position));
		      	}

		      	void OnSaveInstanceState(Bundle state){
		      		choiceMode.onSaveInstanceState(state);
		      	}

		      	void onRestoreInstanceState(Bundle state){
		      		choiceMode.onRestoreInstanceState(state);
		      	}

		      	int getCheckedCount(){
		      		return(choiceMode.getCheckedCount());
		      	}
                 
                void clearChecks(){
                	choiceMode.clearChecks();
                }

                @Override
                public void onViewAttachedToWindow(T holder){
                	super.onViewAttachedToWindow(holder);

                	if(holder.getAdapterPosition!=choiceMode.getCheckPosition()){
                		((RowController)holder).setChecked(false);
                	}
                }

		      }
		
为了这么做，onChecked()问ChoiceMode它是不是单选的。如果是，它就获取了最后一个选中的。如果位置是合理的，它就通过findViewHolderForAdapterPosition()获取
RecyclerView.ViewHolder。如果它返回的不是null的话，它就是一个RowController，所以调用setChecked(false)的时候会移除激活状态。


findViewHolderForAdapterPosition()和findViewHolderForLayoutPosition（）代替了现在不赞成的findViewHolderForPosition（）方法。三个方法都做着相同基本的
一件事：给定一个位置，如果有的话返回这个位置的ViewHolder。至少目前来说，findViewHolderForAdapter()和findViewHolderForLayoutPosition()有着相同的实现。
findviewHolderForAdapterPosition最大的不同是在数据改变的时候总是返回空（例如，notifyDataSetChanged()被调用了。）但是这些改变还没有铺放。在这个样例中，差别是学理上的,但是在大都数用例中findViewHolderForAdapter()可能是更安全的选择。

但是呢，这些find...（）方法有个问题，它们只在行可见的时候返回ViewHolder。如果ViewHolder被缓存了，但不可见的话，find...（）仍旧不会返回它。这就导致了一个问题，
当我们需要反选一个不可见行的时候,是不使用onBindViewHolder重绑定的。（因为ViewHolder已经设置好了).这就需要我们实现onViewAttachedToWindow（）-在ViewHolder内容真正作为子视图副驾到RelativeLayout上是调用，并且更新选择状态作为反馈。


setChecked()在之前的样例中不存在，因为激活状态完全由RowController内部处理。所以呢，现在RowController现在有个setChecked()方法来切换激活状态：

	void setChecked(boolean isChecked){
		row.setActivated(isChecked);
	}

MainActivity现在必须在onCreate()中把RecyclerView供给IconicAdapter:

	IconicAdapter(RecyclerView rv){
		super(rv,new SingleChoiceMode());
	}


视觉上来说，除了一次只能选中一个之外，结果是跟之前的样例一模一样的。关键的是至多这个短语，这个实现允许用户轻触一个选中的并进行反选。这可能很好，因为你的应用在这个情形中
简单的隐藏了细节，仍旧允许用户和action bar项交互。(例如一个“创建新的模型”项)。如果你想阻止这个，当用户轻触之情选中的项不要让SingleChoiceMode把checkedPosition
设为－1，保持当前选中的位置不同。

####Action Modes

ListView给予我们还有就是支持action modes。特别的，“multiple－choice modal"设置会自动开始和结束一个action mode。

再一次的，RecyclerView没有action modes的钩子。尽管如此，你可以自己去实现。我们要手动启动和销毁aciton mode，此外还要反馈aciton mode上用户的交互。（轻触项，或者手动退去action mode)

变得有趣的是选中项和action mode之间联系。这里有两个用户体验规定：

1. 没有项选中的时候，应该没有action mode

2.没有action mode的时候，应该没有选中项

你可能认为，这两个规定是一样的，在某种程度上来说它们是的。这样措辞是为了强调状态改变的复杂性：

* 当用户选中一个项，action mode应该出现
* 当用户反选最后一个选中的项，因此没有更多选中的项了，aciton mode应该消失
* 当用户退去action mode(例如，按下返回键)，所有的项应该反选

处理这些转变是需要花费一点时间的，正如在RecyclerView／ActionModeList中所展示的那样。这是早前ChoiceList样例的一个复制版，当有一个项选中的时候就出现
action mode。action mode的逻辑大部分是从这本书的action 磨的样例中复制过来的，在这个样例中，我们允许用户大写或移除选中的项。

再一次的，我们对ChoiceMode做了一些修改，添加了两个方法：

1. getCheckedCount（）,返回选中的项的数量，被用于作为action mode的副标题

2.clearChecks()，反选所有选中的项

	package com.commonsware.android.recyclerview.actionmodelist;

	import android.os.Bundle

	public interface ChoiceMode{
		void setChecked(int position,boolean isChecked);
		boolean isChecked(int position);
		void onSaveInstance(Bundle state);
		void onRestoreInstanceState(Bundle state);
		int getCheckedCount();
		void clearChecks();
	}

MultiChoiceMode 实现了这些，外加对setChecked（）的细微修改。之前版本的MultiChoceMode只是简单地把状态的布尔值放在ParcelableSparseBooleanArray中，默认为false。
现在我们移除了没有选中的项，所以ParcelableSparseBooleanArray中只有选中的项。这使得getCheckedCount()和clearChecks()实现变得很简单：

	package com.commonsware.android.recyclerview.actionmodelist;

	import android.os.Bundle;

	public class MultiChoiceMode implements ChoiceMode{
		private static final String STATE_CHECKS_STATES="checkStates";
		private ParcelableSparseBooleanArray checkStates=new ParcelableSparseBooleanArray();
        
        @Override
        public void setChecked(int position,boolean isChecked){
        	if(isChecked){
        		checkStates.put(position,isChecked);
        	}
        	else{
        		checkStates.delete(position);
        	}
        }

        @Override
        public boolean isChecked(int position){
        	return(checkStates.get(position,false));
        }

        @Override
        public void onSaveInstanceState(Bundle state){
        	state.putParcelable(STATE_CHECK_STATES,checkStates);
        }


        @Override
        public void onRestoreInstanceState(Bundle state){
        	checkStates=state.getParcelable(STATE_CHECK_STATES);
        }

        @Override
        public int getCheckedCount(){
        	return(checkStates.size());
        }

        @Override
        public void clearChecks(){
        	checkStates.clear();
        }

	}


ChoiceCapableAdapter暴露了两个新的ChoiceMode功能给它的子类

    package com.commonsware.android.recyclerview.actionmodelist;

    import android.os.Bundle;
    import android.support.v7.widget.RecyclerView;

    abstract public class 
         ChoiceCapableAdapter<T extends RecyclerView.ViewHolder>
         extends RecyclerView.Adapter<T>{
          private final ChoiceMode choiceMode;

          public ChoiceCapableAdapter(ChoiceMode choiceMode){
            super();
            this.choiceMode=choiceMode;
          }

          void onChecked(int position,boolean isChecked){
            choiceMode.setChecked(position,isChecked);
          }

          boolean isCheck(int position){
            return (choiceMode.isChecked(position));
          }

          void onSaveInstanceState(Bundle state){
            choiceMode.onSaveInstance(state);
          }

          void onRestoreInstanceState(Bundle state){
            choiceMode.onRestoreInstanceState(state);
          }

          int getCheckCount(){
            return(choiceMode.getCheckCount());
          }

          void clearChecks(){
            choiceMode.clearChecks();
          }

         }


IconicAdapter现在不仅继承了ChoiceCapableAdapter,它也实现了ActionMode.Callback接口，负责管理aciton mode:

    class IconicAdapter extends ChoiceCapableAdapter<RowController>
          implements ActionMode.Callback{

IconicAdapter现在重载了onChecked()，通常只由ChoiceCapableAdapter处理。除了链接父类的标准行为，IconicAdapter还负责管理action mode：

  * 如果我们选择一个项（isChecked is true），并且我们没有正在进行的action mode,使用startActionMode
  * 如果我们有选中的项，并且我们已经有了action mode了，只需要更新副标题，因为我们选中项的数据变了。
  * 如果我们没有任何选中项，并且我们有一个激活的action mode,结束这个action mode，因为反选最后的一个选中的时候，action mode 就不需要了。

		@Override
		void onChecked(int position,boolean isChecked){
		  super.onChecked(position,isChecked);

		  if(isChecked){
		     if(activeMode==null){
		       activeMode=startActionMode(this);
		     }
		     else{
		        updateSubTitle(activeMode);
		     }
		  }
		  else if(getCheckedCount()==0&&activeMode!=null){
		      activeMode.finish();
		  }
		} 

因为IconicAdapter实现了ActionMode.Callback,它需要通过接口实现方法。包括:

* onCreateActionMode(),设置action mode
* onPrepareActionMode()，仅仅因为它被接口需要
* onActionItemClicked(),真正需要做工作的地方，但是现在呢先写个TODO注释
* onDestroyActionMode(),确保所有的项不选中(clearChecks())并告诉RecyclerView.Adapter数据已经改变了，强制
  重新绘制所有的可见行，会影响不再选中的。

 		@Override
 		public boolean onCreateActionMode(ActionMode mode,Menu menu){
 		   MenuInfalter infalter=getMenuInflater();

           infalter.infalte(R.menu.context,menu);
           mode.setTitle(R.string.context_title);
           activeMode=mode;
           updateSubtitle(activeMode); 

           return (true);
        } 
        @Override
        public boolean onPrepareActionMode(Action mode,Menu menu){
           return (false);
        }

        @Override
        public boolean onActionItemClicked(ActionMode mode,MenuItem item){
           //TODO:do something based on the action

           updateSubTitle(activeMode);
           return(true);
        }

        @Override
        public void onDestoryActionMode(ActionMode mode){
          if(activeMode!=null){
             activeMode=null;
             clearChecks();
             notifyDataSetChanged();
          }
        }

 updateSubTilte()方法，引用之前的一些方法，只更新了action mode的副标题来反映
 当前选中的项的数量：

        private void updateSubtitle(ActionMode mode{
           mode.setSubTitle("("+getCheckedCount()+")");
        }

应用结果跟原本的ChoiceList样例代码大部分一样，直到我们选中或更多的项的时候，action mode 出现了：


![wizard](/the_busy_coder's_guide/img/figure_446.png)

###改变内容

之前样例的显而易见的问题是我们为没有响应用户在action mode item上的点击做任何事情。我们真的应该大写或移除单词。但是，这
就包含了修改模型数据和模型数据被RecyclerView展现的方式。


不是很明显的问题在action mode消失的时候我们调用了notifyDataSetChanged(),强制全部重新绘制RecyclerView的内容。虽然这可行，
但是有点小题大做，因为只有部分可见的项被选中了。理想的情况是，在actionmode完成且为未选中的时候，我们应该只更新我们当前选中的某些位置。
就像在single-choice list样例中一样，我们可以使用findViewHolderByPosition（）,找到收到影响的RowController实例。但是实际上，
更新选中状体是另一种大写或移除单词造成相同问题的表现形式，我们需要确保RecyclerView按理想的最小工作量去描述当前的模型状态。

####更新已存在的内容

使用基于BaseAdapte的AdapterView和Adapter,位置告诉AdapterView模型数据改变的方法是notifyDataSetChanged().这会触发整个
AdapterView的重建，是缓慢且昂贵的。

虽然RecyclerView.Adapter也有它自己的notifyDataSetChanged()，那只是用于重新加载全部模型数据的，例如从数据库获取新的游标或者不明确改变是什么的时候。
当你深入UI内部的变化，并且特别是如果你使用的模型数据是像ArrayList这样的模型对象的时候，你可以使用在RecyclerView上使用比notifyDataSetChanged()更具细粒度的方法。

如果某一项被更新了-例如单词大写-你可以在RecyclerView上使用notifyItemChanged()来指出特定位置已经改变了。可选方案包括:

* notifyItemMoved（）,来只是一个项还在模型数据中但是处于一个新的位置
* notifyItemRangeChanged(),指示一个范围内的位置已经修改了，而不是重复调用notifyItemChanged()

当用户大写单词的时候，ActionModeList2使用notifyItemChanged()，如果需要的话让这些项重新绘制。如果一个或多个当前在RecyclerView中不可见的话，可能不是立即需要。

但是目前位置，我们的模型数据一致是静态的字符串数组，并且我们需要一个可变的模型。所以使用和ListView action mode样例相同的方法，把静态数组转化为ArrayList。

MainActivity现在的成员变量是字符串数组，使用的是ORIGINAL_ITEMS这个静态数组：

    private static final String[] ORIGINAL_ITEMS={"lorem","ipsum","dolor",
     "sit","amet",
     "consectetuer","adipiscing","elit","morbi","vel",
     "ligula","vitae","arcu","aliquet","mollis",
     "emtiam","vel","erat","placerat","ante",
     "porttitor","sodales","pellentesque","augue","purus"
     };
    private ArrayList<String> items;

使用引用的项现在使用私有的getItems()方法来懒实例化列表：

    private ArrayList<String> getItems(){
        if(items==null){
          items=new ArrayList<String>();
          for(String s: ORIGINAL_ITEMS){
            items.add(s);
          }
        }
        return(items);
    }

我们同样需要在配置改变的时候持有这些项，因为这些项可能被用户改变。除了原本的让ChoiceCapableAdapter持久化选中状态之外    
，在MainActivity中的我们的onSaveInStanceState()和onRestoreInstanceState()方法来处理这个工作。

    @Override
    public void onSaveInstanceState(Bundle state){
      adapter.onSaveInstanceState(state);
      state.putStringArrayList(STATE_ITEMS,items)；
    }

    @Override
    public void OnRestoreInstance(Bundle state){
      adapter.onRestoreInstance(state);
      items=state.getStringArrayList(STATE_ITEMS);
    }

(这里，STATE_ITEM是一个静态的成员变量，作为Bundle入口的常量键值)

为了能大写化或移除列表中的选中的单词，我们需要知道哪一个选中了。为了不直接暴露数据，ChoiceMode现在有了一个
visitChecks()方法，通过这个方法给每个选中的位置提供一个Vistor:

    package com.commonsware.android.recyclerview.actionmodelist2;

    import android.os.Bundle;

    public interface ChoiceMode{
      void setChecked(int position,boolean isChecked);
      boolean isChecked(int position);
      void onSaveInstanceState(Bundle state);
      void onRestoreInstanceState(Bundle state);
      int getCheckedCount();
      void clearChecks();
      void visitChecks(Visitor v);
    }
     
MultiChoiceMode 通过遍历checkStates ParcelableSparseBooleanArray实现了visitChecks()。
这样的话，如果visitor修改了选中状态，我们的循环是不受影响的。

    @Override
    public void visitChecks(Vistor v){
      SparseBooleanArray copy=checkStates.clone();
      for(int i=0;i<copy.size();i++){
        v.onCheckedPosition(copy,keyAt(i));
      }
    }

visitChecks()和其它ChoiceMode中的方法一样，visitChecks()也由ChoiceCapableAdapter暴露的。

现在，IconicAdapter可以使用visitChecks（）大写化单词:
    
    case R.id.cap:
        visitChecks(new ChoiceMode.Vistor(){
           @Override
           public void onCheckedPosition(int position){
              String word=getItems().get(position);

              word=word.toUpperCase(Locale.ENGLISH);
              getItems().set(position,word);
              notifyItemChanged(position);
           }
        });
        break;
这里呢，对于每个选中的项，我们都对它进行了大写化，把原来的替换成对应的大写的，并且调用notifyItemChanged()来让RecyclerView知道
这个位置的模型数据变了，如果需要的话进行重新绘制。

在onDestroyActionMode()中我们也使用了visitChecks()，来避免调用notifyDataSetChanged（）:

    @Override
    public void onDestoryActionMode(ActionMode mode){
        if(activeMode!=null){
          activeMode=null;
          visitChecks(new ChoiceMode.Vistor（){
            @Override
            public void onCheckedPosition(int position){
              onChecked(position,false);
              notifyItemChanged(position);
            }
          });
        }
    } 

每个选中的都进行反选，并且使用notifyItemChanged()来确保需要时的重新绘制。
 
现在，选中一些项目并且选择action mode中的“CAPITALIZE”会把这些词大写化：

####添加和移除项

在RecyclerView.Adapter上也有你添加或从adapter移除项的时候调用的方法。不仅会造成RecyclerView的自我更新，如果相关的位置是可见的话也会有改变的动画。

具体的，你可以调用:

* notifyItemInserted(),来指示新的项被插入到特定的位置，表中其它的向后移动一个位置。
* notifyItemRangeInserted(),插入一些项到区间
* notifyItemRemoved(),来指示有一项从表中移除了，之后的项移到之前的位置
* notifyItemRangeRemoved()，移除某一个区间中的项

ActionModeList2样例使用notifyItemRemoved()作为处理action mode项目的一部分:
      
      case R.id.remove:
        final ArrayList<Integer> positions=new ArrayList<Integer>();

        visitChecks(new ChoiceMode.Visitor(){
          @Override
          public void onCheckedPosition(int position){
            positions.add(position);
          }
        });

        Collections.sort(positions,Collections.reverseOrder());

        for(int position:positions){
          getItems().remove(position);
          notifyItemRemoved(position);
        }

        clearChecks();
        activeMode.finish();
        break;

因为滑动的时候，移除项的时候项会补位，先移除最低的项是很重要的，然后自底向上。这就是这份代码写成这样的原因：

* 统计列表中选中的位置
* 对列表进行颠倒排序
* 遍历选中的项目】，从ArrayList中移除并调用notifyItemRemoved()来通知adapter原来位置的项没了
* 清除所有选中的项（因为所有的项都移除了）并且结束action mode(因为没有选中的项了)

结果是当用户移除项的时候，它们很快淡出，之后的向上滑动补位。如果你想使用其它动画，你也可以通过创建你自己的RecyclerView.ItemAnimator
并通过使用setItemAnimator（）附加。

###事情的顺序

Version 22+以上的recyclerview-v7提供了SortedList。表面上，这个类看起来提供排序的列表。但是，它有着一个设计成和RecyclerView绑定的回调
接口，所以呢对SortList作出的改变可以影响RecyclerView本身，连带动画和可选的批处理过程等等。

这在RecyclerView/SortedList样例项目中做了阐释。同时，我们会看到如何在fragment中使用RecyclerView以及如何从后台线程生成RecyclerView.

####Gradle的改变
这个项目需要22或者以上的Recyclerview，因为原先的发布的recyclerView-v7没有SortedList。所以呢，Gradle构建文件需要进行一定修改：

    compile 'com.android.support:recyclerview-v7:22.2.0'
    compile 'com.android.support:cardview-v7:22.2.0'

注意cardview-v7的版本要和recyclerview一致。通常来说，最好保持AndroidSupport 类库包同步，至少在主版本上要这样。相似的，它最好
有着对应主版本类库的compileSdkVersion,因为不同类库API不尽相同。因此，这个项目把compileSdkVersion(和buildTools)设成v22:

    compileSdkVersion 22
    buildToolsVersion "22.0.1"


####RecyclerViewFragment

这章之前的内容都是使用一个RecyclerViewActivity作为基础的RecyclerView设置。但是，在此例中，我们要使用一个retained fragment来管理AsyncTask,
建议的是把RecyclerView放进fragment，而不是直接由activity管理。

所以在这个项目中要把RecyclerViewActivity改写成RecyclerViewFragment:
    
    package com.commonsware.android.recyclerview.sorted;

    import android.app.Fragment;
    import android.os.Bundle;
    import android.support.v7.widget.RecyclerView;
    import android.view.LayoutInfalter;
    import android.view.View;
    import android.view.ViewGroup;

    public class RecyclerViewFragment extends Fragment{
      @Override
      public View onCreateView(LayoutInflater inflaterm,ViewGroup container,
                           Bundle savedInstanceState){
       RecyclerView rv=new RecyclerView(getActivity());
       rv.setHasFixedSize(true);
       
       return(rv);
      }
   
      public void setAdapter(RecyclerView.Adapter adapter){

      }

      public RecyclerView.Adapter getAdapter(){
        return(getRecyclerView().getAdapter());
      }

      public void setLayoutManager(RecyclerView.LayoutManager mgr){
        getRecyclerView().setLayoutManager(mgr);
      }

      public RecyclerView getRecyclerView(){
        return((RecyclerView)getView());
      }
    }

大致上说，就是把设置RecyclerView从onCreate()中的移到了onCreateView()。其余的核心API没有改变。

项目有一个SortedFragment,它继承了RecyclerViewFragment并对数据进行加载-我们会在这章之后的内容讲解更多。

修改过的MainActivity修改之后仅仅是通过FragmentTransaction加载SortedFragment。

    package com.commonsware.android.recyclerview.sorted;

    import android.app.Activity;
    import android.os.Bundle;

    public class MainActivity extends Activity{
      @Override
      public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);

        if(getFragmentManager().findFragmentById(android.R.id.content)==null){
          getFragmentManager().beginTransaction()
                             .add(android.R.id.content,new SortedFragment()).commit();
        }
      }
    }    

###SortedFragment

大部分的SortedFragment让人回想起线程那种的AsyncTask演示样例，混合了之前这章的CardView/RecyclerView。SortList就这么产生了。

####SortedList

原先AsyncTask样例中的数据模型是一个简单的ArrayList.现在它是一个SortedList，在SortedFragment的onCreate()
方法中进行初始化：

      @Override
      public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setRetainInstance(true);

        model=new SortedList<String>(String.class,sortCallback);

        task=new AddStringTask();
        task.execute();
      }

SortedList构造器有两个参数:

* 表示列表中模型的Java类对象（此例中，String.class)
* 基于ListAPI(例如add()，insert(),remove（）)的数据改变时会被调用的SortedList.Callback对象

还有第三个容积参数，没有在本例中使用。

我们会很快过一下叫作sortCallback的SortedList.Callback实现。

####IconicAdapter
之前RecyclerView样例直接使用静态数组。现在，我们使用的是模型SortedList.因此，
onBindViewHolder()和getItemCount()需要在之前基础上修改成引用model上合适的方法：
  
      class IconicAdapter extends RecyclerView.Adapter<RowController>{
        @Override
        public RowController onCreateViewHolder(ViewGroup parent,int viewType){
          return(new RowController(getActivity().getLayoutInflater()
                           .inflate(R.layout.row,parent,false)));
        }
        @Override
        public void onBindViewHolder(RowController holder,in position){
          holder.bindModel(model.get(position));
        }

        @Override
        public int getItemCount(){
          return(model.size());
        }

      }

也需要注意的是，我们在onViewCreate()中创建了adapter,adapter是fragment中的一个成员变量:

      @Override
      public void onViewCreated(View view,Bundle savedInstanceState){
        super.onViewCreated(view,savedInstanceState);

        setLayoutManager(new LinearLayoutManager(getActivity()));
        adpter=new IconicAdapter();
        setAdapter(adapter);
      }


####SortedList.Callback

SortedList.Callback的任务是作为SortedList和RecyclerView.Adapter之间的桥梁。SortedList,如其名，对内容进行排序。
这就意味着任何对SortedList内容的改变可以对RecyclerView有不同的影响。例如，添加到ArrayList智慧在RecyclerView末尾添加一行，
添加到SortedList则会添加到中间，来维持排序。

因此，你的SortedList.Callback负责两件事：

* 通过比较元素的方式进行排序
* 传递排序结果到RecyclerView.Adapter，它随后就可以做合适的移动和动画。

谨记以上，sortedCB实现了SortedList.Callback:
    private SortedList.Callback<String> sortcallback=new SortedList.Callback<String>(){
        @Override
        public int compare(String o1,String o2){
           return o1.compareTo(o2);
      }

      @Override
      public boolean areContentsTheSame(String oldItem,String new Item){
        return(areItemsTheSame(oldItem,new Item));
      }

      @Override
      public void onInserted(int position,int count){
        adapter.notifyItemRangeInserted(position,count);
      }

      @Override
      public void onRemoved(int position,int count){
        adapter.notifyItemRangeRemoved(position,count);
      }

      @Override
      public void onMoved(int fromPosition,int toPosition){
        adapter.notifyItemMoved(fromPosition,toPosition);
      }

      @Override
      public void onChanged(int position,int count){
        adapter.notifyItemRangeChanged(position,count);
      }

    }

第一个是你标准的compare()比较方法，你可能在一个Comparator上实现。从某个排序观点来说如果两个模型对象相等就应该返回0，负数表示第一个参数在第二个参数之前，
正数表示第一个参数在第二个参数之后。

然后就有两个命名相似的方法作为equals()的替代在其中你会有个Comparator的是：areContentsTheSame()和areItemsTheSame()。

在两个传入的值代表同一个实际逻辑项的时候，areItemsTheSame()应该返回true。在SortedFragment这个例子中，仅仅是字符串相等不相等。但是对于更为复杂的数据模型，
你应该比较主键或其他类型的不变的标识。

在项的视觉表现看起来一样的时候，areContentsTheSame（）应该返回ture,因为这个可以用于RecyclerView的优化。

举个例子，假定购物车fragment想要使用SortedList。再假定你添加三袋洗衣粉到购物车中，在RecyclerView中是用三行而不是在一行中标上数量3。
在此例中：
* compare()返回一个值来指示这些物品的排序规则，可能是基于项的标题
* areItemsTheSame（）会返回false，因为这三个是逻辑上不同的行
* areContentsTheSame()会返回true，因为虽然这三个是独立的，它们的内容看起来是一样的。

在很多情形下，areContentsTheSame（）仅仅调用areItemsTheSame()就可以了，在不同项可能有不同的视觉表现这个假定下。这个就是这个样例中所做的，areItemsTheSame()
换而使用compare（）来看项之间是否相同。

最后，四个on...（）方法仅仅转发到它们的RecyclerView.Adapter的对应部分，所以对SortedList的改变是对应部分RecyclerView内容改变了。

注意到，SortedListAdapterCallback把RecyclerView.Adapter作为构造器参数并为你处理on...()方法。但是，因为我们想要保存在配置改变的时候
保持SortedList,并且因为SortedList不允许我们改变SortedList.Callback对象，我们不能毫不犹豫地在配置更改之后把SortedList切换到新的fragment和新的adapter。

####AsyncTask
除了现在单词是添加到SortedList的之外，AddStringTask和原本Async样例中的一样，通过SortedList的回调会更新RecyclerView:
    
    private class AddStringTask extends AsyncTask<Void,String,Void>{
      @Override
      protected Void doInBackground(Void... unused){
        for(String item:items){
          if(isCancelled()){
            break;

            publishProgress(item);
            SystemClock.sleep(400);
          }
          return(null);
        }
        
        @Override
        protected void onProgerssUpdate(String...item){
          if(!isCancelled()){
            model.add(item[0]);
          }
        }

        @Override
        protected void onPostExecute(void unused){
          Toast.makeText(getActivity(),R.string.done,Toast.LENGTH_SHORT).show();
          task=null;
        }

      }
    }


####结果
如果你运行这个样例的话，你会看到每400ms都会有单词添加到列表中。但是，在原本的基于ListView的样例中，并且ListView空间
填满之后你是看不到新的行出现的。在此例中，latin单词由SortedList进行排序，并且你会在它们添加的时候动画进入到合适的位置。
最后，你会得到一个基于CardView的RecyclerView实现，除了单词是被排序过的：

![wizard](/the_busy_coder's_guide/img/figure_448.png)

###其他的好东西
引用著名美国专题广告牌的话：”等一下，还有呢!"

除了LinearLayoutManager和GridLayoutManager之外，还有StaggeredGridLayoutManager.使用竖直滚动的GridLayoutManager的话，
行都是一样高的，但是宽度是多变的。使用竖直滚动的StaggeredGridLayoutManager，列都是一样高的，但是高度可能不一样。

三个标准的layout manager同样也支持横向操作，只要改下构造器的一个布尔值就可以。在这些情形中，内容是横向滚动的而不是竖直的。这就不需要第三方
类库像ListView那样的实现了。

并且，当然，你可以实现你自己的Recycler.LayoutManager，而不使用任何内置的。

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

结果是实线的蓝色分隔线:****

![wizard](/the_busy_coder's_guide/img/figure_449.png)

当然，通过这个类库，你可以通过他的建造者样式的API改变更多而不只是它的颜色。









        





