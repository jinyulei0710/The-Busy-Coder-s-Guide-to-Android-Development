###针对音频的媒体播放器

如果你想要播放音乐，特别是MP3格式的，你会想要使用MediaPlayer类。有了它，你可以提供给它一个音频片段，
启动/停止/暂停/回访，并在关键事件得到通知，例如片段准备好播放或者已经播完的时候。

你有三种方式来是指一个媒体播放器并告诉他播放什么音频剪辑：
* 如果剪辑是一个原始的资源，使用MediaPlayer.create()并提供剪辑的资源ID
* 如果你有一个剪辑的Uri，使用Uri方式版本的Media.create()
* 如果你有一个剪辑的字符串路径，就使用默认的构造器创建一个MediaPlayer.然后调用
  setDataSource()设置剪辑的路径。
  
然后，你需要调用prepare()或prepareAsync()。两者都让剪辑准备好播放，例如获取文件或者流的
前几秒的。prepare()方法是同步的，只要它返回了，剪辑就可以播放了。prepareAync()方式是异步的-
稍后才能使用。

一旦这个剪辑准备好了，start()开始播放，pause()停止播放(start()在pause()停止的地方启动)，
stop()停止播放。有一个告诫：你不能简单的在MediaPlayer上调用stop()之后再次调用start()。之后我们会在这节中涵盖到一个应急系统。


为了实战，看一下[Media/Audio](https://github.com/commonsguy/cw-omnibus/tree/master/Media/Audio)这个样例项目，布局是很普通的，三个分别有着play，pause，stop标签的按钮。


	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
 	 <LinearLayout
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="4dip"
 	 >
    <ImageButton android:id="@+id/play"
      android:src="@drawable/play"
      android:layout_height="wrap_content"
      android:layout_width="wrap_content"
      android:paddingRight="4dip"
      android:enabled="false"
    />
    <TextView
      android:text="Play"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:gravity="center_vertical"
      android:layout_gravity="center_vertical"
      android:textAppearance="?android:attr/textAppearanceLarge"
    />
 	 </LinearLayout>
 	 <LinearLayout
   	 android:orientation="horizontal"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:padding="4dip"
     >
     <ImageButton android:id="@+id/pause"
       android:src="@drawable/pause"
       android:layout_height="wrap_content"
       android:layout_width="wrap_content"
       android:paddingRight="4dip"
     />
     <TextView
       android:text="Pause"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:gravity="center_vertical"
       android:layout_gravity="center_vertical"
       android:textAppearance="?android:attr/textAppearanceLarge"
    />
     </LinearLayout>
     <LinearLayout
   	  android:orientation="horizontal"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:padding="4dip"
    >
    <ImageButton android:id="@+id/stop"
       android:src="@drawable/stop"
       android:layout_height="wrap_content"
       android:layout_width="wrap_content"
       android:paddingRight="4dip"
    />
   	 <TextView
       android:text="Stop"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:gravity="center_vertical"
       android:layout_gravity="center_vertical"
       android:textAppearance="?android:attr/textAppearanceLarge"
    	/>
 	 </LinearLayout>
	</LinearLayout>

Java,当然就变得有趣了：

     package com.commonsware.android.audio;

	import android.app.Activity;
	import android.app.AlertDialog;
	import android.content.Context;
	import android.content.SharedPreferences;
	import android.media.MediaPlayer;
	import android.os.Bundle;
	import android.view.Menu;
	import android.view.MenuItem;
	import android.view.View;
	import android.widget.ImageButton;
	import android.widget.Toast;

	public class AudioDemo extends Activity
  		implements MediaPlayer.OnCompletionListener {
  
  		private ImageButton play;
        private ImageButton pause;
        private ImageButton stop;
        private MediaPlayer mp;

      @Override
     public void onCreate(Bundle icicle) {
       super.onCreate(icicle);
       setContentView(R.layout.main);
    
       play=(ImageButton)findViewById(R.id.play);
       pause=(ImageButton)findViewById(R.id.pause);
       stop=(ImageButton)findViewById(R.id.stop);
    
       play.setOnClickListener(new View.OnClickListener() {
          public void onClick(View view) {
          play();
        }
      });
    
      pause.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
           pause();
       }
      });
    
     stop.setOnClickListener(new View.OnClickListener() {
       public void onClick(View view) {
         stop();
       }
     });
    
     setup();
    }
  
      @Override
      public void onDestroy() {
        super.onDestroy();
    
      if (stop.isEnabled()) {
         stop();
       }
     }
  
     public void onCompletion(MediaPlayer mp) {
        stop();
     }
  
     private void play() {
      mp.start();
    
      play.setEnabled(false);
     pause.setEnabled(true);
     stop.setEnabled(true);
    }
  
    private void stop() {
     mp.stop();
     pause.setEnabled(false);
     stop.setEnabled(false);
    
    try {
      mp.prepare();
      mp.seekTo(0);
      play.setEnabled(true);
    }
    catch (Throwable t) {
      goBlooey(t);
    }
    }
  
    private void pause() {
    mp.pause();
    
    play.setEnabled(true);
    pause.setEnabled(false);
    stop.setEnabled(true);
  	}
  
   	private void loadClip() {
    	try {
    	  mp=MediaPlayer.create(this, R.raw.clip);
     	 mp.setOnCompletionListener(this);
    }
    	catch (Throwable t) {
      goBlooey(t);
    	}
  	}
  
 	 private void setup() {
    	loadClip();
    	play.setEnabled(true);
    	pause.setEnabled(false);
    	stop.setEnabled(false);
  	}
  
  	private void goBlooey(Throwable t) {
    	AlertDialog.Builder builder=new AlertDialog.Builder(this);
    
    	builder
      	.setTitle("Exception!")
      	.setMessage(t.toString())
      	.setPositiveButton("OK", null)
      	.show();
  	}
	}

在onCreate()中，我们用三个按钮来接通合适的回调，然后调用setup()。在setup()中，我们创建了我的MediaPlayer，设置成播放我们打包项目raw资源的一个剪辑。我们也配置activity本身作为完成的监听，所以我们在剪辑播放完的时候触发。注意，因为我们使用的是MediaPlayer()的静态create()方法，我们已经显式调用了prepare(),所以我们不需要自己单独调用。

按钮简单地使用了MediaPlayer并通过恰如其分的回调触发了每一个的状态。所以，play()启动了MediaPlayer的播放，pause()暂停了播放，以及stop()停止了播放并重设我们的MediaPlayer让它再次播放。stop()回调也被用作音频剪辑自己完成播放的时候。

要重置MediaPlayer，stop()回调在已存的MediaPlayer上调用了prepare()来让其能再次播放并调用
seekTo()移动到开始的地方。如果我们使用一个外部文件作为我们的媒体资源，他会能更好地调用
prepareAsync()。


界面没有什么特别的，但是我们队样例中的音频更有兴趣：

####流的限制

对于流媒体你能够使用相同的基础代码，使用一个http://或者rtsp://URL。但是，记住Android并不支持RTSP上的MP3流，因为它超出了相关的RTSP规格。也就是说，存在RTSP流上的MP3，并且客户端和服务器协商出了一个点对点的扩展来让规格能接受这个。Android不能播放这些流。


	
	
  


