###你的首个动态碎片

静态fragment是相当简单的，一旦你有了Fragment实现：只要把<fragment>元素添加到你的activity布局中你希望
fragment出现的地方就可以了。

这尽管简单，但是带来了一些开销。我们会在下一章分析其中的一些限制。

这些限制可以通过使用动态碎片从而被克服。你会使用一个FragmentTransaction在代码运行时中把碎片添加上去，而不是在布局中使用<fragment>元素。

有了这个概念之后，来看下Fragments/Dynamic这个样例项目。

这和之前的静态fragment项目是一样的，只不过这次我们把OtherActivity调整成使用动态fragment，具体来说就是一个ListFragment。

####ListFragment类

ListFragment之余fragment就像ListActivit之余activity。为了便于使用，它对ListView进行了封装。所以，
把OtherActivity先放到一边，我们从一个是ListFragment的OtherFragment开始，就像之前例子一样在其中展示我们最喜欢的25个拉丁单词。

就像ListActivity没有调用setContentView()一样，ListFragment并没有必要重载onCreteView()。默认情况下，
整个fragment将会由单个ListView组成。和ListActicity有一个setListAdapter()方法来把适配器和ListView联系起来一样，ListFragment也有相同的方法：

      package com.commonsware.android.dfrag;

      import android.app.Activity;
      import android.app.ListFragment;
      import android.os.Bundle;
      import android.util.Log;
      import android.view.View;
      import android.widget.ArrayAdapter;

      public class OtherFragment extends ListFragment {
      private static final String[] items= { "lorem", "ipsum", "dolor",
          "sit", "amet", "consectetuer", "adipiscing", "elit", "morbi",
           "vel", "ligula", "vitae", "arcu", "aliquet", "mollis", "etiam",
           "vel", "erat", "placerat", "ante", "porttitor", "sodales",
          "pellentesque", "augue", "purus" };

      @Override
       public void onViewCreated(View view, Bundle savedInstanceState) {
         super.onViewCreated(view, savedInstanceState);

         setListAdapter(new ArrayAdapter<String>(getActivity(),
                    android.R.layout.simple_list_item_1, items));
       }


我们在onViewCreated()中调用了setListAdapter()，因为我们知道ListView现在已经可用了。

这个类也重载了很多fragment生命周期方法，记录它们的结果，类似于我们另一个Fragment和LifecycleLoggingActivity。

####Activity类

现在，OtherActivity不再需要加载一个布局了，我们把res/layout/other.xml从项目彻底移除了。取而代之的
，我们会使用一个FragmentTransaction来我们的fragment添加到UI上:

	package com.commonsware.android.dfrag;

    import android.os.Bundle;

    public class OtherActivity extends LifecycleLoggingActivity {
      @Override
      public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

      if (getFragmentManager().findFragmentById(android.R.id.content) == null) {
         getFragmentManager().beginTransaction()
                          .add(android.R.id.content,
                               new OtherFragment()).commit();
        }
      }
    } 
  
  
为了使用FragmentTransaction,你需要FragmentManager。这个对象知道所有存在于你activity的fragment。
如果你使用的是原生API等级为11的版本，你可以通过调用getFragmentManager()获取你的Fragment。如果你使用的
是Android支持包，你需要转而使用getSupportFragmentManager()。


我们在我们的FragmentTransaction调用了两个方法：add()和commit()。add()方法，和你的可能猜测的一样，指示
我们添加一个fragment到UI上。我们提供了真实的fragment对象，在此例中是通过创建一个新的OtherFragment。我们
也需要指示我们要把这个fragment放在哪个位置。在我们记载布局之后，我就可以把这个fragment抛到任意目标容器中。
在我们的示例中，因为我们没有加载一个布局，我们提供了android.R.id.content作为持有我们fragment的视图的容器的
ID。这里，android.R.id.content标识了setContentView()结果将要进入的容器-它是一个由Activity本身所提供的容器并充当我们内容的最高层级容器的角色。

仅仅调用add()是不够的。我们然后需要调用commit()使事务真正发生。

你可能会奇怪为什么我们在真正创建fragment之前会尝试在我们的FragmentManager中找一个fragment。
我们这么做是为了应对配置改变的问题，并且我们会在下一章进行进一步探索。



 
    

