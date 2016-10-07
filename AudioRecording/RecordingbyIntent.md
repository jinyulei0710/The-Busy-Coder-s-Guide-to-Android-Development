###通过Intent进行录制

就像使用设备内置的相机拍照时最简单的一样，最简单录制一些音频的方式是使用内置的activity。同时，和使用内置的相机应用，内置的音频录制activity有着一些明显的限制。

请求内置的音频录制activity是调用startActivityForResult()用于一个MediaStore.Audio.Media.RECORD_SOUND_ACTION。你可以在Media/SoundRecordIntent样例项目中看到，特别是在MainActivity中：

		package android.app.Activity;
		
		import android.app.Activity;
		import android.content.Intent;
		
		import android.os.Bundle;
		import android.provider.MediaStore;
		import android.widget.Toast;
		
		public class MainActivity extends Activity{
		    private static final int REQUEST_ID=1337;
		    
		    @Override
		    public void onCreate(Bundle saveInstaceState){
		        Intent i=new Intent(MediaStore.Audio.RECORD_SOUND_ACTION);
		        
		        startActivityForReuslt(i,RESULT_ID);
		    }
		    
		    @Override
		    protected void onActivityResult(int requestCode,int resultCode,
		                            Intent data){
		         if(requestCode==REQUEST_ID&&resultCode==RESULT_OK){
		              Toast.makeText(this,"Recording finished!",Toast.LENGTH_LONG)
		              .show();
		         }
		         
		         finish();
		    }                        
		}

和这本书中其他少数其它几个样例应用一样，Media/SoundRecordIntent使用的是Theme.NoDisplay activity,回避开了它自己的UI。取而代之的，在onCreate()，我们直接调用了用于MediaStore.Audio.Media.RECORD_SOUND_ACTION。这
会打开录制的activity:

如果用户通过“录制”和停止按钮录制了一些视频，你会在onActivityResult()拿回控制权，
其中你传入了一个Intent，它的Uri(通过getData())会指向MediaStore中的这个音频录制。

但是：

* 你没有文件存在哪里和文件名的控制权。默认情况下这些文件会被扔在外部存储的根目录。
* 你没有音频录制方式的控制权，例如编解码和比特率。例如，默认情况下文件是录制成AMR格式的。
* ACTION_VIEW 可能不能播放这个音频（至少在一些设备上失败了）。不管是因为编解码还是
数据在MediaStore中存放的方式，或是Android上默认音频播放器的限制，不是很明确。

		