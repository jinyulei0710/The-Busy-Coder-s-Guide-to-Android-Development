###`AsyncTask`的替代方案

android中存在着不使用`AsyncTask`处理后台线程的其它方法:

* 你可以使用一个Handler，它有着一个会在主应用线程，处理从后台线程发送而来的消息对象
的handleMessage()方法。

* 你可以提供一个在主应用线程执行的Runnable来在任一视图上进行post(),或在Activity上
调用runOnUiThread()。

* 你可以提供一个Runnable,外加毫秒级的延迟时间，在任一视图上进行postDelayed()，
在至少几毫米过去之后再在主应用线程上运行Runnable。

这些当中，Runnable选项是最易使用的。

这些也能用来让主应用线程来推迟工作，稍后在主应用线程上完成。例如，你可以使用
postDelayed()设置一个轻量级不需要额外的线程开销的轮询循环，例如
由Timer和TimeTask创建的。要了解这是怎么工作的，让我们瞧一瞧[Threads/postDelayed](https://github.com/commonsguy/cw-omnibus/tree/master/Threads/PostDelayed)
这个样例项目。

这个项目包含了单个叫做postDelayedDemo的activity:

    package com.commonsware.android.post;

    import android.app.Activity;
    import android.os.Bundle;
    import android.view.View;
    import android.widget.Toast;

    public class PostDelayedDemo extends Activity implements Runnable {
      private static final int PERIOD=5000;
      private View root=null;

      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.main);
          root=findViewById(android.R.id.content);
        }

        @Override
      public void onResume() {
        super.onResume();

        run();
        }

        @Override
        public void onPause() {
          root.removeCallbacks(this);

            super.onPause();
          }

          @Override
          public void run() {
            Toast.makeText(PostDelayedDemo.this, "Who-hoo!", Toast.LENGTH_SHORT)
              .show();
              root.postDelayed(this, PERIOD);
              }
          }

我们想要每隔5秒钟显示一个Toast。为了这么做，在onCreate()中，我们通过findViewById()获取了
我们的被称为android.R.id.content处理单个activity的UI的容器。然后，在onResume()中，
我们调用了我们activity的run()方法，它展示了Toast然后调用了postDelayed()来进行自我调度(以Runnable的实现的方式)。
当我们的activity位于前台的时候，作为结果Toast每PEROID毫秒之后都会出现。
一旦有别的东西来到的前台，例如用户按下了返回键——我们的onPause()方法
被调用了，其中我们调用了removeCallbacks()来撤销postDelayed()的调用。
