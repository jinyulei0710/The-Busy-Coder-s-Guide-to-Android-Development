###处理点击事件

但是，有个漂亮的分割线并没有解决更大的问题，对输入的响应。

这个Recyclerview本身，除了滚动没有做任何跟输入有关的事情。任何处理用户点击和触发某种类型的反馈是Recyclerview中的视图处理的,例如列表样式中的行。

这样有这样的好处。像RatingBar这样的可点击控件，在ListView的行中一直跟行本身的点击事件冲突。让行能点击的同时，也让行里的内容也能点击就有点棘手了。使用RecyclerView,你能更加显式地控制这类事情是如何处理的，因为你是那个设置所有on－click处理逻辑的人。

当然，这没有给用户很大帮助。用户并不关心什么代码负责输入。用户只想要提供输入。如果你展现给他们竖直滚动的列表样式的用户界面，他们会尝试在列表中的行进行点击病期待某种类型的输出。

RecyclerView的实现方式，意味着很大程度上你要依靠自己处理输入。这就需要更多的代码，在一个理想的世界，会提供一个马上就能用的recyclerview选项。

####响应点击

本质上，响应点击事件是设置一个OnClickListener到合适的视图上的问题。

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

在此例中，因为在行中的控件没有交互造成点击事件的消耗，用户可以在行的任意的地方进行点击，Toast都会出现。如果你遇到更复杂的情形，例如核对清单中－你可以自己决定如何处理行中不同部分的点击事件。我们马上会在之后这章中看到清单了。

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

