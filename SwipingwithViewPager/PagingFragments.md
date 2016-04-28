###碎片翻页

使用`ViewPager`的最简单方式是让它基于用户的滑动在屏幕上对碎片进行翻页。

为了看这个的实际效果，这节会分析下ViewPager/Fragments这个样例项目。

为了能使用`ViewPager`这个项目依赖了Android支持包。在Android Studio中，就是build.grdle文件中
dependencies闭包中的一个compile声明。

###Activity布局

被activity所使用的布局恰好包含了ViewPager。注意，因为ViewPager没有在android.widget包中，我们需要在元素中
放完全限定的类名：

<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.view.ViewPager xmlns:android:="https://schemas.android.com/apk/res/android"
 android:id="@+id/pager"
 android:layout_width="match_parent"
 android:layout_height="match_parent">

注意ViewPager在IDE图形设计的拖放区域是没有的，可能是因为它来自于Android支持包，因此不是所有的项目都会用到。

###Activity

就如你看到的，ViewPagerFragmentDemoActivity本身是及其小的：
	
	package com.commonsware.android.pager;

    import android.app.Activity;
    import android.os.Bundle;
    import android.support.v4.view.ViewPager;

    public class ViewPagerFragmentDemoActivity extends Activity {
      @Override
      public void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        ViewPager pager=(ViewPager)findViewById(R.id.pager);

        pager.setAdapter(new SampleAdapter(getFragmentManager()));
      }
    } 

我们所有做的就是加载布局，通过findViewById()取得ViewPager,然后通过setAdapter()提供SampleAdapter给
ViewPager。

###PagerAdapter

我们的SampleAdapter是从FragmentPager继承而来的并且实现了两个需要的回调方法:

* getCount(),指示ViewPager中的页数
* getItem()，返回ViewPager特定位置的碎片

	    package com.commonsware.android.pager;

        import android.app.Fragment;
        import android.app.FragmentManager;
        import android.support.v13.app.FragmentPagerAdapter;

        public class SampleAdapter extends FragmentPagerAdapter {
           public SampleAdapter(FragmentManager mgr) {
              super(mgr);
           }

         @Override
           public int getCount() {
            return(10);
         }

         @Override
        public Fragment getItem(int position) {
         return(EditorFragment.newInstance(position));
        }

这里，我们说总共有10页，每一个会是一个EditorFragment对象的实例。在此例中，我们使用的是newInstance()工厂方法，而不是使用构造器。至于根本原因会在下一节进行分析。

####碎片

EditorFragment会持有一个全屏的EditText控件，给用户输入输入一大段的文章，
就像在res/layout/editor.xml资源中定义的那样:

    <EditText xmlns:android:"http://schemas.android.com/apk/res/android"
     android:id="@+id/editor"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     android:inputType="textMultiLine"
      android:gravity="left|top"
     />

我们想在ViewPager内部传递碎片的位置编号，仅仅是在用户如何东西之前自定义展示在Edittext的暗示信息。         
使用通常的Java对象的时候，你可能通过构造器传递这个值，但是在fragment上实现构造器却不是个好主意。作为代替的，
诀窍是创建一个静态的工程方法（通常命名为newInstance()）,这个方法会创建fragment并通过更新fragment的“arguments”来提供参数给它。

 
    static EditorFragment newInstance(int position) {
       EditorFragment frag=new EditorFragment();
       Bundle args=new Bundle();

       args.putInt(KEY_POSITION, position);
       frag.setArguments(args);

       return(frag);
     }
    
你可能会好奇为什么我们要这么麻烦去用这个Bundle，而不是使用常规的数据成员。Bundle参数是我们“保存的实例状态”的一部分，是用来处理像平面旋转之类的情形的-只是这本书之后会讲到的概念。当前来说，记住这是一个好的做法就行了。

在onCreateView()中我们inflate了我们的R.layout.editor资源，从中获取EditText,从我们的参数中获取我们的位置，格式化包含位置信息的暗示信息(使用字符串资源)，然后在EditText上设置暗示信息：
  
 
       @Override
      public View onCreateView(LayoutInflater inflater,
                           ViewGroup container,
                           Bundle savedInstanceState) {
      View result=inflater.inflate(R.layout.editor, container, false);
      EditText editor=(EditText)result.findViewById(R.id.editor);
      int position=getArguments().getInt(KEY_POSITION, -1);

       editor.setHint(String.format(getString(R.string.hint), position + 1));

       return(result);
      } 

####结果

当初始化启动的时候，应用展示的是首个碎片:

但是，你可以水平滑动到下一个碎片:

滑动在两个方向都是可以的，只要在你所要的方向存在另一个页面就行。







 
 

    