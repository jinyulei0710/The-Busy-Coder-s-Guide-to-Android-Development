###游标呢？

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
