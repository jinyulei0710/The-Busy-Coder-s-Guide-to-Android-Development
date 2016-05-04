###事件总线

事件驱动编程已经出现快1/4个世纪了。Android的UI模型中的大部分都是事件驱动的，我们可以通过
回调方法(例如,onCreate()是用于“启动一个activity”事件的)和注册监听
(例如，onClickListener是用于用户点击到一个控件上的时候的)找到这些事件。

但是，最初的时候，Android并没有一个开发者能够使用的十分优雅的事件或消息总线。
Intent系统的工作方式像一个消息总线，但是它是旨在跨进程通讯和进程内通讯的，并且有一定的开销。

但是，随着事件的推移，特别是在2012年初的时候，事件总线开始出现了，并且对组织Android应用
内部的通讯和跨线程十分有用。使用得当的话，一个事件总线就能使AsyncTask和其他用于主应用来回
通讯的解决方案变得不需要了，同时对独立代码的逻辑解耦也是有帮助的。

####事件总线是什么?

不管你把它当成一个“事件总线”(或“消息总线”),发布/订阅模式，或观察者模式的一个子集,
一些组件产出事件的同时其它的进行消耗是现代软件开发中的相当常见的编程模式。

事件总线是为了将事件来源和这些事件的消费者解耦开来而设计的。或者，像事件总线的作者说明的：

我想要以一个简单的，集中化的方式来通知那些对特定事件类型有兴趣的代码，同时这些事件的发送
没有存在发布事件代码和接收事件代码的之间任何直接耦合。

使用传统的Java监听或者观察者模式实现，产生事件的组件需要直接接触事件的消费者。
有时候，消费者名单是被限制到一个消费者的，就像许多和Android控件相关的事件处理者一样。
但是这种源控制汇的编码方式限制了灵活性，因为它需要明确注册事件的生产者和消费者。
此外，这些组件之间的直接连接相对来说是一个强耦合，并且大部分时候我们的对象需要时松耦合的。

事件总线提供了一个标准的通讯渠道，事件生产者和事件消费者能够挂到其中去。
事件生产者仅仅需要把事件交给总线，总线会直接把这些事件交给相关的消费者。
这就降低了生产者和消费者之间的耦合，有时候甚至会减少源和汇这些事件的代码量。

####好的,但是为什么我们要烦扰这个问题?

随后，我们会有一些除我们activity之外的组件。特别是，我们会有service，这个组件主要是为了在后台执行
一些操作而设计的。就像activity之间的通讯是松耦合的一样，activity和service之间的通讯也要是松耦合的。
事件总线是让应用其它部分知道某个工作已经完成的一个绝佳的方式。

####介绍`greenrobot`的事件总线

在这个教程中我们会使用的事件总线实现是`greenrobot`的事件总线，一个基于Guava项目的事件总线的
开源实现。使用了`greenrobot`的事件总线，从应用的一个地方发送到应用中另一个毫不相干的另一个地方就
变得相当简单了。

为了举例说明它的使用，让我们看下EventBus/AsyncDemo这个样例项目。这是之前那个使用AsyncTask来模拟
下载我们的拉丁单词列表样例的重写版，当这些单词来的时候填充进ListView。这个样例用一个
会保存单词记录模型碎片替代了AsyncTask，并且会有一个后台线程下载单词。我们会使用由模型碎片
发起的事件来让UI碎片知道单词到达了。

#####定义事件

有了`greenrobot`的事件总线，“事件”就成了你所能定义的任意对象类。每个不同的类代表一个不同的类型
的事件，并且你可以定义如你所愿一样多的事件类。这些类不需要继承任何特殊的基类，或实现一些特定的接口，
或任何不可思议的注解。它们仅仅是类。

你可能希望把数据成员，构造器，和访问方法放到事件类中去，用来传递你想要传递给事件本身的数据。例如，一个
SearchEvent，可能把搜索查询字符串作为事件的对象的一部分包含进来。

在我们的例子中，我们有一个包含新单词的WordReadyEvent:

      package com.commonsware.android.eventbus;

       class WordReadyEvent {
          private String word;

          WordReadyEvent(String word) {
            this.word=word;
           }

          String getWord() {
            return(word);
           }
        }

#####发布事件

要发布一个事件，你所需要做的是获取一个EventBus实例-通常通过在EventBus上的getDefault()方法
获取-并在其上调用post()，传入要投递到你应用中所有对这个事件感兴趣的部分。

记住了这一点，让我们看下会加载我们单词的ModelFragment：

      package com.commonsware.android.eventbus;

      import android.app.Fragment;
      import android.os.Bundle;
      import android.os.SystemClock;
      import java.util.ArrayList;
      import java.util.Collections;
      import java.util.List;
      import de.greenrobot.event.EventBus;

      public class ModelFragment extends Fragment {
          private static final String[] items= { "lorem", "ipsum", "dolor",
          "sit", "amet", "consectetuer", "adipiscing", "elit", "morbi",
          "vel", "ligula", "vitae", "arcu", "aliquet", "mollis", "etiam",
          "vel", "erat", "placerat", "ante", "porttitor", "sodales",
          "pellentesque", "augue", "purus" };
          private List<String> model=
          Collections.synchronizedList(new ArrayList<String>());
          private boolean isStarted=false;

          @Override
          public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setRetainInstance(true);

            if (!isStarted) {
              isStarted=true;
              new LoadWordsThread().start();
            }           
            }

            public ArrayList<String> getModel() {
              return(new ArrayList<String>(model));
            }

            class LoadWordsThread extends Thread {
              @Override
              public void run() {
                for (String item : items) {
                  if (!isInterrupted()) {
                    model.add(item);
                    EventBus.getDefault().post(new WordReadyEvent(item));
                    SystemClock.sleep(400);
                  }
                }
              }             
            }
          }

这个碎片没有UI，它的存在仅仅是为了管理宿主activity的数据模型。因此这个碎片中，没有
onCreateView()或其它与UI逻辑直接相关的。

在onCreate()中，我们会调用setRetainInstance(true),那样的话如果用户旋转了屏幕
或以其它方式触发了配置改变，我们的模型碎片会保存这个改变并附加到新的activity实例上去。然后，
如果我们没有启动LoadWordsThread，实际上我们是这么做的。LoadWordsThread遍历了我们的单词列表,
休眠了400ms来模拟真正的工作，添加每一个单词到它所管理一个ArrayList上..并调用post()发起一个
WordReadyEvent()让其它的东西知道模型已经改变了。

#####接收事件

为了接收发出的事件，你需要做三件事情:

1.在EventBus上调用register()告诉它你有一个想要接收事件的对象

2.在EventBus上调用unregister()告诉它停止传送事件到之前已经注册的对象

3.实现onEventMainThread()，或其它onEvent()风格方法，来指示你所想要接收的事件类型(并
  实际处理这些事件)

这个样例应用有着一个执行这三个步骤的AsyncDemoFragment:

      package com.commonsware.android.eventbus;

      import android.app.Activity;
      import android.app.ListFragment;
      import android.os.Bundle;
      import android.view.View;
      import android.widget.ArrayAdapter;
      import java.util.ArrayList;
      import de.greenrobot.event.EventBus;

      public class AsyncDemoFragment extends ListFragment {
        private ArrayAdapter<String> adapter=null;
        private ArrayList<String> model=null;

        @Override
        public void onViewCreated(View view, Bundle savedInstanceState) {
          adapter=
          new ArrayAdapter<String>(getActivity(),
                     android.R.layout.simple_list_item_1,
                     model);

                     getListView().setScrollbarFadingEnabled(false);
                     setListAdapter(adapter);
                   }                  

                   @Override
                   public void onAttach(Activity activity) {
                     super.onAttach(activity);

                     EventBus.getDefault().register(this);
                    }

                    @Override
                    public void onDetach() {
                      EventBus.getDefault().unregister(this);

                        super.onDetach();
                        }

                        public void onEventMainThread(WordReadyEvent event) {
                          adapter.add(event.getWord());
                        }                       

                        public void setModel(ArrayList<String> model) {
                            this.model=model;
                            }
                }


碎片以重载onViewCreated()开始，其中我们创建了一个ArrayAdapter并使用它来填充
ListView。

onAttach()和onDetach()方法是我们指示我们想要fragment对象接收相关发送了的事件的地方。
onAttach()调用register()；onDetach()调用unregister().

onEventMainThread()方法通过它的参数，指示我们对已经被发送了的WordReadyEvents感兴趣。
我们的onEventMainThread()方法会在每个WordReadyEvent()事件被传递到EventBus的时候
被调用。就像方法名所暗示的，onEventMainThread()是在主线程上被调用的，所以对我们的UI的
更新是安全的。`greenrobot`的EventBus是负责把这个事件带回主应用线程的-注意我们是从
LoadWordsThread发送事件的，它是一个后台线程。

在onEventMainThread()中，我们获取了新添加的单词，我们可以把这个单词加到我们的ArrayAdapter
上。ArrayAdapter上的add()附加单词到了列表的末尾并通知所联系的ListView数据以及
变了，所以ListView本身要重绘。

从这个类中的代码看我们在onViewCreated()获取model的方式是不一定的。AsyncDemoFragment有着
它自己的单词列表，通过setModel()方法进行设置。我们的ArrayAdapter封装了这个模型。但是单词的
主拷贝是由ModelFragment持有的。如果ModelFragment有了模型，并且AsyncDemoFragment需要
模型，这两个东西是如何联系到一起的呢？

#####Activity

这是由我们的宿主activity处理的，因为它设置了这两个碎片:

      package com.commonsware.android.eventbus;

      import android.app.Activity;
      import android.app.FragmentManager;
      import android.app.FragmentTransaction;
      import android.os.Bundle;

      public class AsyncDemo extends Activity {
        private static final String MODEL_TAG="model";
        private ModelFragment mFrag=null;

        @Override
        public void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);

          FragmentManager mgr=getFragmentManager();
          FragmentTransaction trans=mgr.beginTransaction();

          mFrag=(ModelFragment)mgr.findFragmentByTag(MODEL_TAG);

            if (mFrag == null) {
                mFrag=new ModelFragment();
                trans.add(mFrag, MODEL_TAG);
            }

            AsyncDemoFragment demo=
                (AsyncDemoFragment)mgr.findFragmentById(android.R.id.content);

                  if (demo == null) {
                    demo=new AsyncDemoFragment();
                    trans.add(android.R.id.content, demo);
                  }

                  demo.setModel(mFrag.getModel());

                  if (!trans.isEmpty()) {
                    trans.commit();
                    }
                    }
                  }

在onCreate()中，我们首先看的是我们是否已经有了一个被FragmentManager的MODEL_TAG下持有的
模型实例。如果没有的话，我们会创建一个ModelFragment的实例并通过一个FragmentTransaction把
它添加到FragmentManager上的这个TAG下。

我们然后看我们是否已经有了一个AsyncDemoFragment的实例。如果没有的话，我们会创建一个并
把它添加到FragmentManager上，通过另一个FragmentTransaction把它的UI注入到android.
R.id.content。

然后，我们连接了两个碎片，在ModelFragment上调用getModel()并把结果交给
了AsyncDemoFragment上的setModel()。

当我们的activity是新启动的，没有一个存在的碎片。两个碎片都是都是创建的，
并且AsyncDemoFragment从ModelFragment获取它的模型数组。数组最初是空的。
当ModelFragment添加数据到数组，它传送了WordReadyEvent，这会触发AsyncDemoFragment
告诉ArrayAdapter和ListView模型数据以及改变了。

如果我们经历了一次配置改变，ModelFragment被保留了，但是AsyncDemoFragment却没有。
因为，activity会一直创建一个AsyncDemoFragment.但是我们给AsyncDemoFragment的模型
可能已经有数据了，并且这些单词会在ArrayAdapter封装model的时候就立即出现了。
如果LoadWordsThread仍旧在运行，新的AsyncDemoFragment会捡起所有新发起的WordReadyEvent,
触发它去像之前那样更新ListView。
