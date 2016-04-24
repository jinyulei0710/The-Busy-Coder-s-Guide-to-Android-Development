###Extra!Extra!

有时候，你可能想把一些数据从一个activity传到另一个activity。例如，我们可能有一个展示
我们模型对象集合(例如，书籍)的ListActivity并且我们有着一个用来单独展示特定模型对象的DetailActivity。
DetailActivity需要以某种方式知道要展示哪个模型对象。

一种完成这个工作的方式是通过意图的`extras`。

Intent上有一系列的putExtra()方法，来允许你把键/值对数据打包进Intent中。虽然你不能传入任意的对象，
大都数的基本数据类型是支持的，例如字符串和一些列表类型。下一节内容，会分析更多关于什么能放进Intent的extra。

所有的activity都可以调用getIntent()来取得用来把它启动的意图，然后可以调用多种形式的get...Extra（...指示的是数据类型）来获取所有打包的`extra`。

例如，让我们看下Activities/Extras这个样例项目.

这几乎是之前这章的Activities/Explict的一个复制版。但是，这次，我们的第一个
activity会传递一个extra给第二个:

    package com.commonsware.android.extra;

    import android.app.Activity;
    import android.content.Intent;
    import android.os.Bundle;
    import android.view.View;

    public class ExtrasDemoActivity extends Activity {
      @Override
      public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
         setContentView(R.layout.main);
       }
  
    public void showOther(View v) {
       Intent other=new Intent(this, OtherActivity.class);
    
       other.putExtra(OtherActivity.EXTRA_MESSAGE, getString(R.string.other));
       startActivity(other);
       }
    }   
  
我们和之前一样创建了Intent,然后调用了putExtra()，提供了一个键(一个静态的叫做OtherActivity.EXTRA_MESSAGE字符串)和一个值(R.string.other字符串资源)。然后，只有这样，我们才能调用startActivity()。

我们修改的OtherActivity然后获取extra,同时inflate了TextView(通过findViewById())并把文本填充进去:

    package com.commonsware.android.extra;

    import android.app.Activity;
    import android.os.Bundle;
    import android.widget.TextView;

    public class OtherActivity extends Activity {
     public static final String EXTRA_MESSAGE="msg";
  
     @Override
      public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.other);
    
      TextView tv=(TextView)findViewById(R.id.msg);
     
      tv.setText(getIntent().getStringExtra(EXTRA_MESSAGE));
      }
    }

  
视觉上来说，结果是一样的。功能上来说，所展示的文本是从上一个activity传过来的。

  
  
  