###感受异步

处理现场问题的常用做法是使用`AsyncTask`。有了AsyncTask,Android会在一个后台线程中处理
所有的独立工作的协调工作，而不是在UI线程中。此外，Android本身会分配和移除这个后台线程。并且，
它维护了以小的工作队列，强调了`AsyncTask`一劳永逸的感觉。

`Theodore Levitt`曾经说过这么一句关于市场营销的话：“人们要买的不是一个1/4英寸的钻机，它们想要的是
1/4英寸的洞”。五金店不能贩卖洞，所以它们卖的是次最好的办法：很容易做出个洞的设备(钻机和钻头)。

同样的，许多在后台线程管理方面挣扎的开发者，要的不是后台线程，他们要的是让工作不在UI线程完成，从而避免`jank`。虽然Android不能像变魔术那样能让工作不使用UI线程时间，Android却可以提供能使后台线程操作更加简单和显而易见的东西。`AsyncTask`就是其中的一个例子。

为了使用`AsyncTask`，你必须：

1.创建一个AsyncTask的子类

2.重载一个或多个`AsyncTask`的方法来完成后台工作，此外不管什么与任务相关的工作
都需要在UI线程完成(例如更新进度)。

3.在需要的时候，创建一个`AsyncTask`子类的实例然后调用execute来让它开始进行它的工作。

你所不需要做的是:

1.创建你自己的后台线程

2.在适当的时间终止这个后台线程

3.调用所有类型的方法来安排让部分处理在UI线程完成

####`AsyncTask`,泛型和可变参数

创建一个`AsyncTask`的子类并不像实现Runnable接口那样容易。`AsyncTask`使用了泛型，所以
你需要指定三种数据类型:

1.需要被任务处理的信息的类型(例如，所要下载的多个URL)

2.任务内部传递的用来指示进度的信息类型

2.当任务完成时传递给post任务的信息类型

使人更加迷惑的是头两个参数使用的是可变参数，意味着有一个这些类型的数组在你的
`AsyncTask`子类中使用。

当我们就着样例讲的时候会变得清晰。

####`AsyncTask`的几个步骤

为了完成你的目标，你能够在`AsyncTask`中重载四个方法。

一个你必须重载的，对任务类很有用的是`doInBackground`。这个方法
会由AsyncTask在一个后台线程上进行调用。为了完成这个特定任务所需的工作，它会
运行他所需的足够长的时间。注意，尽管如此，任务还是要是有限的，使用无限循环是不推荐的。

`doInBackground()`会以参数的形式接收以上所列三种数据类型中的第一个可变参数数组，需要被任务处理的
数据。所以，如果你任务的使命是下载多个URL的集合的话，`doInBackground`会接收那些URL到进程中去。

`doInBackground()`方法必须返回以上所列的第三个数据类型-后台工作的结果。

你可能希望重载`onPreExecute()`。这个方法是在后台线程执行`doInBackground`之前从UI线程被调用的。
在这里，你可能初始化一个进度条或其它指示后台工作要开始的东西。

同样的，你可能也会想重载`onPostExecute()`。这个方法是在`doInBackground()`完成之后，从UI线程中被
调用中的。它以参数的形式接收了由`doInBackground()`返回的值。这里，你可能会打发走进度条并使用后台的结果，
例如更新列表的内容。

此外，你可能会希望重载`onProgressUpdate()`。如果`doInBackground()`调用了任务的`publishProgress()`方法，
但是传递给这个方法的对象是在UI线程中提供给onProgressUpdate()的。这样的话，onProgressUpdate()就能在后台任务工作完成的时候给予用户以警告。onProgressUpdate()方法会接收以上列出的第二个可变参数数据类型-通过`publishProgress`由`doInBackground()`发布的数据。


####一个样例任务

就像之前提及过的，实现一个AsyncTask不是和实现一个Runnable一样简单的。但是，一旦你
你获得了传过去的泛型和可变参数，它就还不赖。

`AsyncTask`实战，这节会分析Threads/AsyncTask这个样例项目。

#####碎片和它的AsyncTask

我们有一个叫做AsyncDemoFragment的ListFragment：

  package com.commonsware.android.async;

    import android.app.ListFragment;
    import android.os.AsyncTask;
    import android.os.Bundle;
    import android.os.SystemClock;
    import android.view.View;
    import android.widget.ArrayAdapter;
    import android.widget.Toast;
    import java.util.ArrayList;

    public class AsyncDemoFragment extends ListFragment {
    private static final String[] items= { "lorem", "ipsum", "dolor",
        "sit", "amet", "consectetuer", "adipiscing", "elit", "morbi",
        "vel", "ligula", "vitae", "arcu", "aliquet", "mollis", "etiam",
        "vel", "erat", "placerat", "ante", "porttitor", "sodales",
        "pellentesque", "augue", "purus" };
        private ArrayList<String> model=new ArrayList<String>();
        private ArrayAdapter<String> adapter=null;
        private AddStringTask task=null;

        @Override
        public void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);

          setRetainInstance(true);

          task=new AddStringTask();
          task.execute();

          adapter=
            new ArrayAdapter<String>(getActivity(),
                                  android.R.layout.simple_list_item_1,
                                  model);
  	}

    @Override
    public void onViewCreated(View v, Bundle savedInstanceState) {
      super.onViewCreated(v, savedInstanceState);

      getListView().setScrollbarFadingEnabled(false);
      setListAdapter(adapter);
    }

    @Override
    public void onDestroy() {
      if (task != null) {
        task.cancel(false);
      }

      super.onDestroy();
    }

    class AddStringTask extends AsyncTask<Void, String, Void> {
      @Override
      protected Void doInBackground(Void... unused) {
        for (String item : items) {
          if (isCancelled())
            break;

            publishProgress(item);
            SystemClock.sleep(400);
          }

          return(null);
        }

        @Override
        protected void onProgressUpdate(String... item) {
          if (!isCancelled()) {
            adapter.add(item[0]);
          }
        }   

        @Override
        protected void onPostExecute(Void unused) {
          Toast.makeText(getActivity(), R.string.done, Toast.LENGTH_SHORT)
            .show();
            task=null;
          }
        }
      }


这是另一个乱数假文单词列表的变种，经常在这本书中被使用。这次我们的`AsyncTask`实现中，我们模拟在后台使用AddStringTask创建这些单词，而不是简单地把单词类别交给一个ArrayAdapter。

在onCreate()中我们调用了setRetainInstance(true),那样的话在配置改变的时候Android会
保留这个碎片，例如在屏幕翻转的时候。因为我们的碎片是新创建的，我们把我们的模型初始化成了
一个字符串值组成的ArrayList，此外触发我们的`AsyncTask`，保存AddStringTask到一个task数据成员中。
然后，在onViewCreated()中，我们设置了adapter并把它附加到ListView上去，并隐藏了scrollbar。


在AddStringTask的声明中,我们使用泛型来设置我们所要实现的特定数据类型。具体来说：

1.此例中我们不需要任何配置信息，所以我们的首个类型是Void

2.我们想要把每个由我们后台任务生成的字符串传递给`onProgressUpdate()`,
那么我们就能把它添加到我们的列表中，所以我们的第二个类型是字符串

3.严格来说我们没有任何结果，所以我们的第三类型是Void

`doInBackground()`方法是在一个后台线程中被动用的。因此，我们想要执行多久都可以。
在一个应用产品中，我们可能会遍历一个URL列表然后下载每个东西。这里，我们遍历了我们的静态
乱数假文单词列表，在每个上面调用了publishProgress()方法，然后休息400毫秒来模拟真实的工作过程。
在每个传递上我们也调用了isCancelled()，来看我们的任务是否已经被取消了，如果我们已经清理了这个
后台线程的话就跳过这个工作。

因为我们没有选择任何配置信息，在doInBackground()中我们应该不需要参数。但是AsyncTask所使用的契约告诉我们
需要接受一个第一个数据类型的参数变量，这就是为什么我们的方法参数是Void...的原因。

因为我们没有选择任何结果，我们不应该返回任何东西。再一次的，AsyncTask所使用的契约告诉我们要返回第三个数据类型的
对象。因为数据类型是Void,我们返回的对象就是null。

onProgressUpdate()方法是在UI线程中被调用的，我们想要做一些事情让用户
知道我们是在加载这些字符串。在此例中，我们只不过是添加字符串到了ArrayAdapter上，
那么他就附加到了列表的末尾。但是，我们只在任务还没有取消的时候去这么做。

onProgressUpdate()方法接收了一个字符串类型的可变参数，因为它是声明在我们类中的
第二个可变参数。因此我们每次调用publishProgress()的时候只传递一个字符串参数，所以
我们只需要分析可变参数数组的第一个条目。

onPostExecute()方法是在UI线程中被调用的，并且我们想要做一些事情来指示后台工作已经完成了。
在一个真实的系统中，可能会有要打发走的一些进度条或一些要停止的动画。这里，我们仅仅发出一个Toast然后
把task设置成null。我们不需要单词调用isCancelled()，因为onPreExecute()在任务已经被取消的情况下不会被调用。

因为我们没有结果，我们应该不需要任何参数。AsyncTask所使用的契约告诉我们应该接受单个第三个数据类型的值。
因为这个数据类型是Void,我们的方法参数是Void unused。

为了使用AddStringTask，我们只不过创建了一个实例，然后在其上面调用了execute()。
这触发了导致后台线程执行它的工作的事件链。

如果AddStringTask()需要配置参数，我们不会使用Void作为我们的首个数据类型，并且构造器会接收
零个或多个所定义类型的参数。这些值最终会传递到doInBackground()中去。

我们的碎片也有一个如果AsyncTask未完成时(task是非空的)，会在AsyncTask上调用cancel()的onDestroy()方法。
这个取消任务的和检查任何是否已经取消的工作存在是有两个原因的：

1.效率，在我们任务本身不再需要的情况下，我们应该跳过所有不再需要的重要工作。

2.为了避免如果我们尝试在一个已经销毁的activity上发出一个Toast而引发崩溃，
例如用户启动了activity，然后在我们完成后台线程工作并展示Toast之前按下了返回键。

#####Activity和结果

AsyncDemo是带有一个启动动态碎片实例的标准方式的Activity：


    package com.commonsware.android.async;

    import android.app.Activity;
    import android.os.Bundle;

    public class AsyncDemo extends Activity {
        @Override
        public void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);

                if (getFragmentManager().findFragmentById(android.R.id.content) == null) {
                    getFragmentManager().beginTransaction()
                          .add(android.R.id.content,
                              new AsyncDemoFragment()).commit();
                }
              }
            }


如果你构建，安装，并运行了这个项目，你会看到列表每几秒都会被实时生成，同时伴随着指示完成的
Toast。

####线程和配置改变

默认的activity的销毁和创建周期在经历配置改变的时候，后台线程会存在一个问题。
如果activity已经启动了一些后台工作,例如通过一个AsyncTask,然后activity被销毁
和重建，AsyncTask需要以某种方式知道。否则的话，AsyncTask可能会发送更新
和最终结果到旧的activity，新的activity却一无所知。实际上，新的activity
可能会再次启动后天线程，浪费了资源。


这就是为什么，在以上的样例中，我们保存了碎片实例。碎片实例持有者它的数据模型并且
知道不要仅仅是因为配置改变就去触发一个新的AsyncTask。此外，我们保留了数据模型，所以
由于配置改变而重新创建的ListView能使用由旧的数据模型恢复而来的新的adapter，所以我们
不会丢失我们已有的拉丁单词集。

我们从后台线程中尝试引用activity的时候也要十分小心。因为，假定在doInBackground()执行
期间，用户旋转了屏幕。我们所使用的activity在主应用线程上随即会发生改变，独立的工作会在
后台中完成。在配置改变在发生的时候，通过getActivity()返回的activity可能不是处于一个可用的状态。

但是，对我们来说从onPreExecute()甚至从onProgressUpdate()中使用getActivity()都是安全的。
对于这些回调来说，配置改变要么没有发生要么已经完成了，我们不会处于中间状态。

####哪里不要使用AsyncTask

AsyncTask,尤其是和动态碎片结合使用的时候，是满足一个后台线程大部分需求的极好的解决方案。

这句话中的关键字是“大部分”。

`AsyncTask`管理着一个线程池，它从这个池子里拉取用作任务实例的线程。线程池假定在一个合理的
时间段之后它们可以拿回它们的线程。因此，当你不知道你需要线程要运行多长时间的时候，`AsyncTask`
不是一个很好的选择。

####关于`AsyncTask`线程池

此外，`AsyncTask`管理的线程池大小是多变的。

在Android 1.5中，它是单个线程

在Android 1.6中，它扩展到支持很多的并发线程，可能比你需要的还多。

在Android 3.2中，它收缩回到了单个线程，如果你的android:targetSdkVersion
被设置成13或者更高的版本，要解决以下有关的问题:

* Fork太多的线程使CPU供应不足
* 开发者认为fork出来的任务之间有顺序依赖关系，但并发执行的时候是没有顺序可言的。

如果你想从API等级11开始的话，你可以供应你自己的`Executor`(来自java.util.concurrent包)，
它可以提供你所想要的线程池，所以你更能够进行管理。

除了串行之外，一次一个`Executor`，有一个内置的实现了旧线程池的`Executor`，你可以使用
这个，而不用自己去弄。

如果你的`minSdkVersion`是11或者更高，如果你明确地想要选择多线程线程池，就使用`executeOnExcutor`(`AsyncTask.THREAD_POOL_EXECUTOR`)。

如果你的minSdkVersion比11低，单是你仍旧想要这么做..但是只能在API等级11以上的设备上，较旧的设备上就落回到execute()。
这个静态工具方法为你处理了这个：

        @TargetApi(Build.VERSION_CODES.HONEYCOMB)
        static public <T> void executeAsyncTask(AsyncTask<T,?,?>task,
                                T...params){
            if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.HONEYCOMB){
              task.executeOnExcutor(AsyncTask.THREAD_POOL_EXECUTOR,params);
            }
            else{
              task.execute(params);
            }
        }


为了使用这个，调用executeAsyncTask()的时候，传入你的AsyncTask实例以及那些你原本传给execute()
的参数。

这里我们做了一个说明，就@TargetApi注解而言，会在这本书之后讲到。

也要注意到多线程线程池中的线程数量这些年来也是一直变化的。最初的时候，
线程池能最多能有120个线程，这个数量太大了。在Android 4.4的时候，线程池只会增长到
“CPU核的数量*2+1”个，所以在一个双核的设备上，线程池的上线会是5个线程。
更多的任务会排到队列中，最多排到128个队列任务。
