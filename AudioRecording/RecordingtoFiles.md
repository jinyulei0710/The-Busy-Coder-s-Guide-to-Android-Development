###录制成文件

如果你的目标是录制一个音频笔记，一个报告或者与这些类似的东西，然后MediaRecorder可能是你想要的。它会让你指定你想要录制何种类型的media，以什么格式，以及存放在哪。它然后会处理实际的录制工作。

为了阐释这个，让我们分析Media/AudioRecording这个样例应用。

我们activity的布局包含了单个的叫做record的ToggleButton：

		<ToggleButton xmlns:android="http://schemas.android/apk/res/android"
		 xmlns:tools="http://schemas.android.com/tools"
		 android:id="@+id/record"
		 android:layout_width="match_parent"
		 android:layout_height="match_parent"
		 android:textAppearance="?android:sttr/textAppearanceLarge"/>

在MainActivity的onCreate()中，我们加载了了布局，为了能发现用户按下按钮的动作，设置了activity本身作为onCheckChanged的监听者：

          @Override
          public void onCreate(Bundle savedInstance){
             super.onCreate(savedInstanceState);
             setContentView(R.layout.activity_main);
             
             ((ToggleButton)findViewById(R.id.record)).setOnCheckedListener(this);
          }
 同样的在onResume()中，我们初始化了一个MediaRecorder，设置activity去处理录制的info和error事件。类似的，
 我们在onPause()中我们release()了MediaRecorder,来减少当我们不处于前台时的开销：
 			
 			  @Override
 		   public void onResume(){
 		      super.onResume();
 		      
 		      recorder=new MediaRecoder();
 		      recoder.setOnErrorListener(this);
 		      recoder.setOnInfoListener(this)
 		  }        
          
            @Override
            public void onPause(){
               recoder.release();
               recoder=null;
               
               super.onPause();
          }
		 
大多数的工作发生在onCheckedChanged(),当用户按下按钮的时候我们获得了控制权。当我们处于选中状态时，我们开始了录制；如果没有，我们停止前一个录制：


          @Override
          public void onCheckedChanged(CompoundButton buttonView,boolean isChecked){
              if(isChecked){
                  File output=
                   new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS),BASENAME);
                   
                   recoder.setAudioSource(MediaRecorder.AudioSource.MIC);
                   recoder.setOutputFormat(MediaRecoder.OutputFormat.THREE_GPP);
                   recoder.setOutputFile(output.getAbsolutePath());
                   
                   if(Build.VERSION.SDK_INT>=Build.VERSION_CODES_GINGERBREAD_MR1){
                        recorder.setAudioEncoder(MediaRecoder.AudioEncoder.ACC);
                        recorder.setAudioEncodingBitRate(160*1024);
                   }
                   else{
                        recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
                   }
                   recorder.setAudioChannels(2);
                   try{
                      recorder.prepare();
                      recorder.start();
                   }
                   catch(Exception e){
                     Log.e(getClass().getSimpleName(),"Exception in preparing recorder",e);
                     Toast.makeText(this,e.getMessage(),Toast.LENGTH_LONG).show();
                   } 
              }else{
                 try{
                    recorder.stop();
                 }catch(Exception e){
                    Log.w(getClass().getSimpleName(),"Exception in stopping recorder",e);
                    //call fail if start() failed for some reason
                 }
                 recorder.reset();
              }
          }
          
为了录制音频,我们:

* 创建一个文件对象代表录制的音频存放的地方，在此例中使用Environment.getExternalStoragePublicDirectory()来找出外部存储上的位置
* 通过调用setAudioSource()告诉MediaRecorder我们想要从麦克风录制,通过调用setOutputFormat()告诉MediaRecorder()我们要录制一个3GP文件，通过调用setOutputFile()告诉MediaRecorder把录制结果
放到我们的setOutputFile()
* 如果我们运行在Android2.2.3或者更高的版本上，我们也可以通过setAudioEncoder()配置我们的编码为AAC并
通过setAudioEncodingBitRate()设置我们需要的比特率为160Kbs-否则的话，我们使用setAudioEncoder()要求
AMR窄带。
* 通过setAudioChannels()设置我们想要多少个channel，例如录制立体声就要2个。
* 通过调用prepare()(设置输出文件)和record()启动实际的录制

当用户释放按钮的时候停止录制，仅仅是调用MediaRecorder的stop()。

因为我们告诉MediaRecorder我们的activity是Error和Info的监听者，我们在activity上分别实现
它们需要的方法(onError()和onInfo()).在常规事件中，它们不应该被触发。如果发生了，我们传入一个整形，
通常命名为what，指示发生了什么:

                @Override
               public void onInfo(MediaRecorder mr,int what, int extra){
                   String msg=getString(R.string.strange);
                   
                   switch(what){
                      case MediaRecorder.MEDIA_RECORDER_INFO_MAX_DURATION_REACHED:
                      msg=getString(R.string.max_duration);
                      break;
                      case MediaRecorder.MEDIA_RECORDER_INFO_MAX_FILESIZE_REACHED:
                      
                   }
               
               } 

		 