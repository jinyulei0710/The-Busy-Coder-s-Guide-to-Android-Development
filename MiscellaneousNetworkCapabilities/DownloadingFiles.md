###下载文件

Android 2.3引进了一个DownloadManager,是为处理大量下载较大文件的复杂性问题而设计的,例如：

1. 获取用户使用的是WiFi还是移动蜂窝数据信息，知道之后决定下载该不该发生

2. 处理用户，之前在Wifi环境，然后移动到无限访问节点范围之外之后，在移动蜂窝数据上不成功的情况

3. 确保设备在下载过程中保持唤醒状态

DownloadManager 本身比你自身写代码去处理以上这些问题会简单点。但是，它有一些挑战性。在这节中，我们会
检视[Internet/Download](https://github.com/commonsguy/cw-omnibus/tree/master/Internet/Download)这个使用DownloadManager的样例项目。

####权限

为了使用DownloadManager，你会需要持有INTERNET权限。你也会需要WRITE_EXTERNAL_STOREAGE权限，因为
DownloadManager只能下载到外部存储。即使你尝试让DownloadManager在某个可能不需要权限的地方写入（例如，
Android 4.4＋设备上的getExternalFilesDir()）。DownloadManager需要你持有权限，Android framework更是如此，
并且目前，在所有API等级上DownloadManager都需要权限。

例如，这里是Internet/Download 应用的manifest,在里面呢我们请求了这两个权限:

		<?xml version="1.0" encoding="uft-8"?>
		<manifest xmlns:android:http://shcemas.android.com/apk/res/android"
		 package="com.commonsware.android.downmgr"
		 android:versionCode="1"
		 android:versionName="1.0">
		<supports-screens
		  android:anyDensity="true"
		  android:largeScreens="true"
		  android:normalScreens="true"
		  android:smallScreens="true"/>
		 <uses-sdk 
		   android:minSdkVersion="14"
		   android:targetSdkVersion="14"/>
		 <uses-permission android:name="android.permission.INTERNET"/>
		 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

		 <application
		  android:icon="@drawable/ic_launcher"
		  android:label="@string/app_name"
		  android:theme="@android:style/Theme.Holo.Light.DarkActionBar">
           <activity
		     android:name=".DownloadDemo"
		     android:label="@string/app_name">
		     <intent-filter>
		       <action android:name="android:intent.action.MAIN"/>
		       <category android:name="android.intent.category.LAUNCHER"/>
		     </intent-filter>
		     </activity>
		  </application> 
		  </manifest>
####布局

我们的样例应用有一个简单的布局，包含了三个按钮:

1.一个来启动下载

2.一个来询问下载的状态

3.一个来展示由系统提供的包含下载文件清单的activity

		<?xml version="1.0" encoding="uft-8"?>
		<LinearLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         android:orientation="vertical"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         >
         <Button
         android:id="@+id/start"
         android:text="@string/start_download"
         android:layout_width="match_parent"
         android:layout_height="0dip"
         android:layout_weight="1"
         />
         <Button
          android:id="@id/query"
          android:text="@string/query_status"
          android:layout_width="match_parent"
          android:layout_height="0dip"
          android:layout_weight="1"
          android:enabled="false"
          />
          <Button android:id="@+id/view"
           android:text="@string/view_log"
           android:layout_width="match_parent"
           android:layout_height="0dip"
           android:layout_weight="1"
         />
         </LinearLayout>


####请求下载

要开始下载，我们首先要获得DownloadManager。它是一个所谓的“系统服务""。你可以在任意activity
（或其它Context)上调用getSystemService(),提供给它你想要的系统服务的标识，并接受系统服务对象返回。
但是，因为getSystemService()支持大范围的这些对象，你需要把你请求的服务强制转换为适当类型。

所以，例如，这里是DownloadFragment中的onCreateView()方法，从中我们获取了DownloadManager:

	@Override
	public View onCreateView(LayoutInflater,ViewGroup parent,Bundle savedInstanceState){
		mgr=(DownloadManager)getActivity().getSystemService(Context.DOWNLOAD_SERVICE);

		View result=inflater.inflate(R.layout.main,parent,false);

		query=result.findViewById(R.id.query);
		query.setOnClickListener(this);
		start=result.findViewById(R.id.start);

		start.setOnclickListener(this);

		result.findViewById(R.id.view).setOnClickListener(this);

        return(result);
	}		

大都数这些处理者没有close()或release()或goAwayPlease()这种类型的方法－你需要使用它们然后
让垃圾回收来对它们进行清理。

有了manager之后，我们可以调用一个enqueue()方法来请求下载。这个名字是相对的-不要假设你的下载会立即开始，虽然通常它会。
enqueue()方法把DownloadManager.Requests对象作为一个参数。请求对象使用的是建造者模式，其中大都数方法返回的是请求本身，
所以你能用较少的输入把一系列调用链接到一起。

例如，布局顶部的按钮是和DownloadFragment中的一个startDownload()方法绑定的，如下所示:

	 private void startDownload(View v){
	 	Uri uri=Uri.parse("https://commonsware.com/misc/test.mp4");

	 	Environment.getExternalStoreagePublicDirectory(Environment.DIRECTORY_DOWNLOADS)
	 	       .mkdirs();

	 	DownloadManager.Request req=new DownloadManager.Request(uri);

	 	req.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI｜
	 		DownloadManager.Request.NETWORK_MOBILE)
	 		.setAllowedOverRoaming(false)
	 		.setTitle("Demo")
	 		.setDesctiption("Something useful.No,really.")
	 		.setDestionationInExternalPublicDir(Environment.DIRECTORY_DOWNLOADS,
	 			                              "test.mp4");
	    lastDownload=mgr.enqueue(req);
	    
	    v.setEnabled(false);
	    query.setEnabled(true); 			                              
	 }

我们下载了一个样例MP4文件，并且我们要把它下载到外部存储区域。为了完成后者，我们在环境上使用了getExternalStoragePublicDirectory,它
给了我们一个存放某类内容的合适路径。在此例中，我们要把下载的内容存放在Environment.DIRECTORY_DOWNLOADS，尽管如此当我们下载视频片段的时候，也可以选择Environment
.DIRECTORY_MOVIES。注意由getExternalStoragePublicDirectory()返回的File对象可能指向一个还未创建的目录，这就是为什么我们在它之上调用mkdirs的原因，来确保
目录是存在的。

我们然后使用以下属性创建了DownloadManager.Request对象：

1.我们下载的是我们要的一个特定URL，所以把Uri提供给Request构造器

2.我们要使用移动蜂窝数据或WiFi来进行下载(setAllowedNetworkTypes()），但是我们不想
下载会产生漫游费(setAllowedOverRoaming())

3.我们想要文件作为外部存储上的下载区域上的test.mp4下载

我们提供了一个名词(setTitle（）)和描述(setDescription（）),用作这个下载在通知栏中条目的一部分。下载过程中，当用户下滑通知栏的时候，
会看到它们。

enqueue()方法返回了这个下载的一个ID，它是在查询下载状态时使用的。

####记录下载状态

如果用户按下了查询状态按钮，我们想要的是查明下载进行的详情。为了那么做，我们可以在DownloadManager()上调用
DownloadManager。query()方法把DownloadManager.Query对象把DownloadManager.Query作为参数，它是用来描述你
关心的是哪个下载。在我们的情况中，我们使用的是当用户请求下载时从enqueue()方法获取的值：

		private void query(View v){
			Cursor c=mgr.query(new DownloadManager.Query().setFilterById(lastDownload));

			if(c==null){
				Toast.makeText(getActivity(),R.string.download_not_found,Toast,LENGTH_LONG).show();
			}else{
				c.moveToFirst();

    			  Log.d(getClass().getName(),
          			  "COLUMN_ID: "
             			   + c.getLong(c.getColumnIndex(DownloadManager.COLUMN_ID)));
      			  Log.d(getClass().getName(),
                  "COLUMN_BYTES_DOWNLOADED_SO_FAR: "
                   + c.getLong(c.getColumnIndex(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR)));
                   Log.d(getClass().getName(),
                   "COLUMN_LAST_MODIFIED_TIMESTAMP: "
                   + c.getLong(c.getColumnIndex(DownloadManager.COLUMN_LAST_MODIFIED_TIMESTAMP)));
                   Log.d(getClass().getName(),
                   "COLUMN_LOCAL_URI: "
                      + c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI)));
                   Log.d(getClass().getName(),
                   "COLUMN_STATUS: "
                    + c.getInt(c.getColumnIndex(DownloadManager.COLUMN_STATUS)));
                    Log.d(getClass().getName(),
                    "COLUMN_REASON: "
                     + c.getInt(c.getColumnIndex(DownloadManager.COLUMN_REASON)));

                    Toast.makeText(getActivity(), statusMessage(c), Toast.LENGTH_LONG)
                   .show();

                    c.close();
			}
		}	

query()方法返回了一个Cursor，包含了了关于我们下载详情的一系列列。在DownloadManager上类上列出这一系列常量的提纲。
在我们的情形中，我们取得了（然后倾倒到LogCat）:

1. 下载的ID(COLUMN_ID)

2. 已经下载的数据数量(COLUMN_BYTES_DOWNLOADED_SO_FAR)

3. 下载的最近修改时间戳(COLUMN_LAST_MODIFIED_TIMESTAMP)

4. 本地存放的文件的位置(COLUMN_LOCAL_URI)

5. 真实的状态(COLUMN_STATUS)

6. 状态产生的原因(COLUMN_REASION)

注意如果用户在你尝试访问列和文件下载完成之间的时候删除文件的话，COLUMN_LOCAL_URI可能不可用。

有一些可能的状态码(例如..STATUS_FAILED,STATUS_SUCCESSFUL，STATUS_RUNNING)。一些像STATUS_FAILED这个状态码，可能伴随有一个
提供更多信息的原因。


注意，当你完成的时候你应该关闭这个Cursor。例如，StrictMode会在你没有关闭的时候抱怨。

####下载广播

为了查出下载的结果，我们需要注册一个广播，来观察两个被DownloadManager使用的过程：

1.ACTION_DOWNLOAD_COMPLETE,下载完成的时候让我们知道。

2.ACTION_NOTIFICATION_CLICKED,当用户点击了展示在用户设备上的与我们下载相关的通知的时候让我们知道。

所以，在我们fragment()的onResume()方法中，我们在单个BroadCastReceiver上注册了
这两个事件：

		@Override
		public void onResume(){
			super.onResume();

			IntentFilter f=
			       new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE);

			f.addAction(DownloadManager.ACTION_NOTIFICATION_CLICKED);
			
			getActivity().register(onEvent,f);       
		}

BroadcastReceiver()在onPause()中进行了注销():

		@Override
		public void onPause(){
			getActivity().unregisterRegistger(onEvent);

			super.onPause();
		}		

BroadcastReceiver 实现了检查进入Intent的action字符串（通过调用getAction()，要么展示一个Toast（为ACTION_NOTIFICATION_CLICKED)或
启用开始下载按钮：

		public void onReceive(Context ctxt,Intent i){
			if(DownloadManager.ACTION_NOTIFICATION_CLICKED.equal(i.getAction())){
				Toast.makeText(xtxt,R.string.hi,Toast.LENGTH_LONG).show();
			}
			else{
				start.setEnabled(true);
			}
		}

####用户看到了什么

用户启动应用的时候，看到了我们三个可爱的按钮:

![wizard](/the_busy_coder's_guide/img/figure_678.png)

点击首个在进行进行的时候会禁用按钮，并且一个下载图标会出现在状态栏中（因为Android的图标和Android的状态栏的对比不大，有点难看到）：

![wizard](/the_busy_coder's_guide/img/figure_679.png)

下滑通知栏会展示给以进度条控件的形势展示给用户：

![wizard](/the_busy_coder's_guide/img/figure_680.png)

点击通知栏上的条目会返回对我们原本activity的控制，在其中我们看到了，由我们的BroadcastReceiver提出的Toast。

如果在下载的时候点击中间的按钮，一个不同的Toast会出现指示下载正在进行中:

![wizard](/the_busy_coder's_guide/img/figure_681.png)

附加信息也被倾倒到Logcat:

   	  03-28 17:44:17.136 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_ID: 863
   	  03-28 17:44:17.136 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_BYTES_DOWNLOADED_SO_  FAR: 0
      03-28 17:44:17.136 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_LAST_MODIFIED_TIMESTAMP: 1459158115894
 	  03-28 17:44:17.136 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_LOCAL_URI: file:///storage/emulated/0/Download/test.mp4
 	  03-28 17:44:17.146 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_REASON: 0
		

一旦下载完成了，点击中间的按钮会指示关于下载的确实的,完整的和最终的信息被发送到Logcat：

       03-28 17:36:28.136 6411-6411/com.commonsware.android.downmgr I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@42e2d558 time:121535204
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_ID: 862
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_BYTES_DOWNLOADED_SO_FAR: 6217728
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_LAST_MODIFIED_TIMESTAMP: 1459157718824
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_LOCAL_URI: file:///storage/emulated/0/Download/test.mp4
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_STATUS: 8
		03-28 17:36:38.646 6411-6411/com.commonsware.android.downmgr D/com.commonsware.android.downmgr.DownloadFragment: COLUMN_REASON: 0

点击底部的按钮带来了展示所有下载的activity,下载包括成功的和失败的:

![wizard](/the_busy_coder's_guide/img/figure_682.png)

当然，文件是下载的.



####TODO
