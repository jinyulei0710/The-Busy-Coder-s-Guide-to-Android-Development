###改变项

目前为止，所有RecyclerView中的项有着相同的基础结构，只是这些项中控件的内容不同。但是，我们需要基于不同布局的有着实质不同的项也是完全
有可能的。ListView及其亲属通过ListAdapter中的getViewTypeCount()和getItemViewType（）对其进行处理。RecyclerView和RecyclerView.Adapter
提供了一个相似的机制，包括它们自己的getItemViewType（）变种方法。在这节中，我们会检视这个是如何在列表和网格。
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

####网格形式的表格

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
