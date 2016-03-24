###分割线选项

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
