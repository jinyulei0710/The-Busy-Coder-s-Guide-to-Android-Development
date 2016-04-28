###指示器

ViewPager本身，没有包含可见的指示器。在许多情况下，这是很好的，因为页面本身会包含位置的暗示信息。
但是，即使是在这些情形中，用户也不会完全清楚到底有多少也，哪个方向是能有效滑动的。

因此，你可能想要附加其它控件到ViewPager从而能够帮助提示用户它们处于页面空间的哪个位置。

PagerTitleStrip和PagerTabStrip

主要可供选择的内置的指示器有PagetTitleStrip和PagerTabStrip。正如名字所暗示的，PagerTitleStrip是一个展示你页面标题的长条。PagerTabStrip也差不多，但是它的标题是某种形式的页签，并且它们是可点击的(切换到所点击的页面),然后
PagerTitleStrip是非交互式的。

要使用它们中的一个，你首先必须把它添加到你的布局中，你的ViewPager的里面，就像ViewPager/Indicator样例项目中的res/layout/main.xml资源所展示的那样，一个ViewPager/Fragments项目的拷贝，在这个项目中我们在UI中添加了一个PagerTabStrip。

	<?xml version="1.0" encoding="utf-8"?>
    <android.support.v4.view.ViewPager xmlns:android="http://schemas.android.com/apk/res/ android"
	android:id="@+id/pager"
	android:layout_width="match_parent"
	android:layout_height="match_parent">

	<android.support.v4.view.PagerTabStrip
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_gravity="top"/>

      </android.support.v4.view.ViewPager>
	
这里我们把PagerTabStrip的android:layout_gravity设置成了top,所以它出现在页面之上。你样可以把它设置成bottom让它出现在页面之下。

我们的SampleAdapter需要另一个方法:getPageTitle(),这个方法会返回给定位置所要展示的标题:

    package com.commonsware.android.pager2;

    import android.app.Fragment;
    import android.app.FragmentManager;
    import android.content.Context;
    import android.support.v13.app.FragmentPagerAdapter;

    public class SampleAdapter extends FragmentPagerAdapter {
      Context ctxt=null;

      public SampleAdapter(Context ctxt, FragmentManager mgr) {
        super(mgr);
        this.ctxt=ctxt;
     }

     @Override
      public int getCount() {
        return(10);
      }

     @Override
     public Fragment getItem(int position) {
       return(EditorFragment.newInstance(position));
      }

      @Override
      public String getPageTitle(int position) {
       return(EditorFragment.getTitle(ctxt, position));
      }
    }	
	
这里，我们在EditorFragment上调用了一个静态的getTitle()方法。这是对我们之前onCreateView()方法的少许
重构，在其中我们创建了暗示信息所要用的字符串-我们会使用这个暗示文本作为我们的页面标题：

      package com.commonsware.android.pager2;

     import android.app.Fragment;
     import android.content.Context;
     import android.os.Bundle;
     import android.view.LayoutInflater;
     import android.view.View;
     import android.view.ViewGroup;
     import android.widget.EditText;

     public class EditorFragment extends Fragment {
       private static final String KEY_POSITION="position";

        static EditorFragment newInstance(int position) {
           EditorFragment frag=new EditorFragment();
           Bundle args=new Bundle();

           args.putInt(KEY_POSITION, position);
           frag.setArguments(args);

          return(frag);
      }

        static String getTitle(Context ctxt, int position) {
         return(String.format(ctxt.getString(R.string.hint), position + 1));
      }

        @Override
       public View onCreateView(LayoutInflater inflater,
                           ViewGroup container,
                           Bundle savedInstanceState) {
        View result=inflater.inflate(R.layout.editor, container, false);
       EditText editor=(EditText)result.findViewById(R.id.editor);
        int position=getArguments().getInt(KEY_POSITION, -1);

        editor.setHint(getTitle(getActivity(), position));

        return(result);
      }
    }	
    
    
###第三方指示器

除了长条页面标题之外，如果你想要一些其它的用于你的指示器的话，你可能想要看下ViewPagerIndicator这个类库。
这个类库包含了一系列承担和PagerTitle相同职责的不同外观的控件。在本身稍后内容中，我们会看看其中的一个指示器，TabPageIndicator。

###造访小径

这是可能引起你关于兴趣的高级ViewPager技巧的一章。

	
	