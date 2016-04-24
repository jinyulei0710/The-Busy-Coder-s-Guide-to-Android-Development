###漫步生命周期

为了知道这些多个生命周期方法是什么时候被调用的，让我们分析下Activities/Lifecycle这个样例项目.

除了我们的两个activity不是直接继承Activity之外，这个项目是和Activities/Extras项目一样的。
取而代之的我们引进了一个LifecycleLoggingActivity作为基类并让我们的activity继承它：

	 package com.commonsware.android.lifecycle;

    import android.app.Activity;
    import android.os.Bundle;
    import android.util.Log;

     public class LifecycleLoggingActivity extends Activity {
      @Override
        public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    
         Log.d(getClass().getSimpleName(), "onCreate()");
      }
  
      @Override
       public void onRestart() {
         super.onRestart();
    
         Log.d(getClass().getSimpleName(), "onRestart()");
       }
  
        @Override
        public void onStart() {
          super.onStart();
    
          Log.d(getClass().getSimpleName(), "onStart()");
        }
  
       @Override
       public void onResume() {
        super.onResume();
    
         Log.d(getClass().getSimpleName(), "onResume()");
      }
  
      @Override
      public void onPause() {
         Log.d(getClass().getSimpleName(), "onPause()");
    
          super.onPause();
      }
  
       @Override
       public void onStop() {
        Log.d(getClass().getSimpleName(), "onStop()");
    
        super.onStop();
      }
  
       @Override
        public void onDestroy() {
          Log.d(getClass().getSimpleName(), "onDestroy()");
    
            super.onDestroy();
         }
        }  
        
LifecycleLoggingActivity重载了以上提到的生命周期方法并发表了一个指示什么被调用了的调试行。

当我们首次启动应用的时候，我们的首批生命周期方法以我们所预期的顺序被调用了：

04-01 11:47:21.437:D/ExplictIntentsDemoActivity(1473):onCreate()

04-01 11:47:21.827:D/ExplictIntentsDemoActivity(1437):onStart()

04-01 11:47:21.827:D/ExplictIntentsDemoActivity(1437):onResume()

如果我们点击了首个activity上的按钮来启动第二个activity，我们获得了：

04-01 11:47:54.776: D/ExplictIntentsDemoActivity(1473):onPause()

04-01 11:47:54.877: D/OtherActivity(1473):onCreate()

04-01 11:47:54:947: D/OtherActivity(1473):onStart()

04-01 11:47:54:974: D/OtherActivity(1473):onResume()

04-01 11:47:55.347: D/ExplictIntentsDemoActivity(1473):onStop()

注意我们的首个activity是在第二个activity启动之前被暂停的，并且首个activity的onStop()方法被延迟到了第二个
activity出现之后。


如果我们在第二个activity上按下了返回按钮，返回到首个activity，我们看到了：

04-01 11:48:54.807: D/OtherActivity(1473):onPause()

04-01 11:48:54.857: D/ExplictIntentsDemoActivity(1473):onRestart()

04-01 11:48:54.857: D/ExplictIntentsDemoActivity(1473):onStart()

04-01 11:48:54.857: D/ExplictIntentsDemoActivity(1473):onResume()

04-01 11:48:55:257: D/OtherActivity(1473):onStop()

04-01 11:48:55:257: D/OtherActivity(1473):onDestroy()

注意到，进入屏幕发生在离开屏幕的activity的onPause()和的onStop()方法之间。也注意到
在onStop()之后onDestroy()马上就被调用了，因为activity是通过返回按钮结束掉的。


如果我们按下了HOME按钮，让主界面activity来到前台，我们看到了：

04-01 11:50:30.347 D/ExplictIntentsDemoActivity(1473):onPause()

04-01 11:50:32.227 D/ExplictIntentsDemoActivity(1473):onStop()

因为主界面做了它的展示工作，在onPause()和onStop()之间存在一个延迟。并且
没有onDestory(),因为应用仍在运行并且没有东西结束activity。终究，设备会终止我们的进程，
并且如果这是正常发生的，我们会看到onDestory()LogCat消息。








        