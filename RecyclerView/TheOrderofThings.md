###事物的秩序

Version 22+以上的recyclerview-v7提供了SortedList。表面上，这个类看起来提供排序的列表。但是，它有着一个设计成和RecyclerView绑定的回调
接口，所以呢对SortList作出的改变可以影响RecyclerView本身，连带动画和可选的批处理过程等等。

这在RecyclerView/SortedList样例项目中做了阐释。同时，我们会看到如何在fragment中使用RecyclerView以及如何从后台线程生成RecyclerView.

####Gradle的改变
这个项目需要22或者以上的Recyclerview，因为原先的发布的recyclerView-v7没有SortedList。所以呢，Gradle构建文件需要进行一定修改：

    compile 'com.android.support:recyclerview-v7:22.2.0'
    compile 'com.android.support:cardview-v7:22.2.0'

注意cardview-v7的版本要和recyclerview一致。通常来说，最好保持AndroidSupport 类库包同步，至少在主版本上要这样。相似的，它最好
有着对应主版本类库的compileSdkVersion,因为不同类库API不尽相同。因此，这个项目把compileSdkVersion(和buildTools)设成v22:

    compileSdkVersion 22
    buildToolsVersion "22.0.1"


####RecyclerViewFragment

这章之前的内容都是使用一个RecyclerViewActivity作为基础的RecyclerView设置。但是，在此例中，我们要使用一个retained fragment来管理AsyncTask,
建议的是把RecyclerView放进fragment，而不是直接由activity管理。

所以在这个项目中要把RecyclerViewActivity改写成RecyclerViewFragment:
    
    package com.commonsware.android.recyclerview.sorted;

    import android.app.Fragment;
    import android.os.Bundle;
    import android.support.v7.widget.RecyclerView;
    import android.view.LayoutInfalter;
    import android.view.View;
    import android.view.ViewGroup;

    public class RecyclerViewFragment extends Fragment{
      @Override
      public View onCreateView(LayoutInflater inflaterm,ViewGroup container,
                           Bundle savedInstanceState){
       RecyclerView rv=new RecyclerView(getActivity());
       rv.setHasFixedSize(true);
       
       return(rv);
      }
   
      public void setAdapter(RecyclerView.Adapter adapter){

      }

      public RecyclerView.Adapter getAdapter(){
        return(getRecyclerView().getAdapter());
      }

      public void setLayoutManager(RecyclerView.LayoutManager mgr){
        getRecyclerView().setLayoutManager(mgr);
      }

      public RecyclerView getRecyclerView(){
        return((RecyclerView)getView());
      }
    }

大致上说，就是把设置RecyclerView从onCreate()中的移到了onCreateView()。其余的核心API没有改变。

项目有一个SortedFragment,它继承了RecyclerViewFragment并对数据进行加载-我们会在这章之后的内容讲解更多。

修改过的MainActivity修改之后仅仅是通过FragmentTransaction加载SortedFragment。

    package com.commonsware.android.recyclerview.sorted;

    import android.app.Activity;
    import android.os.Bundle;

    public class MainActivity extends Activity{
      @Override
      public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);

        if(getFragmentManager().findFragmentById(android.R.id.content)==null){
          getFragmentManager().beginTransaction()
                             .add(android.R.id.content,new SortedFragment()).commit();
        }
      }
    }    

###SortedFragment

大部分的SortedFragment让人回想起线程那种的AsyncTask演示样例，混合了之前这章的CardView/RecyclerView。SortList就这么产生了。

####SortedList

原先AsyncTask样例中的数据模型是一个简单的ArrayList.现在它是一个SortedList，在SortedFragment的onCreate()
方法中进行初始化：

      @Override
      public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setRetainInstance(true);

        model=new SortedList<String>(String.class,sortCallback);

        task=new AddStringTask();
        task.execute();
      }

SortedList构造器有两个参数:

* 表示列表中模型的Java类对象（此例中，String.class)
* 基于ListAPI(例如add()，insert(),remove（）)的数据改变时会被调用的SortedList.Callback对象

还有第三个容积参数，没有在本例中使用。

我们会很快过一下叫作sortCallback的SortedList.Callback实现。

####IconicAdapter
之前RecyclerView样例直接使用静态数组。现在，我们使用的是模型SortedList.因此，
onBindViewHolder()和getItemCount()需要在之前基础上修改成引用model上合适的方法：
  
      class IconicAdapter extends RecyclerView.Adapter<RowController>{
        @Override
        public RowController onCreateViewHolder(ViewGroup parent,int viewType){
          return(new RowController(getActivity().getLayoutInflater()
                           .inflate(R.layout.row,parent,false)));
        }
        @Override
        public void onBindViewHolder(RowController holder,in position){
          holder.bindModel(model.get(position));
        }

        @Override
        public int getItemCount(){
          return(model.size());
        }

      }

也需要注意的是，我们在onViewCreate()中创建了adapter,adapter是fragment中的一个成员变量:

      @Override
      public void onViewCreated(View view,Bundle savedInstanceState){
        super.onViewCreated(view,savedInstanceState);

        setLayoutManager(new LinearLayoutManager(getActivity()));
        adpter=new IconicAdapter();
        setAdapter(adapter);
      }


####SortedList.Callback

SortedList.Callback的任务是作为SortedList和RecyclerView.Adapter之间的桥梁。SortedList,如其名，对内容进行排序。
这就意味着任何对SortedList内容的改变可以对RecyclerView有不同的影响。例如，添加到ArrayList智慧在RecyclerView末尾添加一行，
添加到SortedList则会添加到中间，来维持排序。

因此，你的SortedList.Callback负责两件事：

* 通过比较元素的方式进行排序
* 传递排序结果到RecyclerView.Adapter，它随后就可以做合适的移动和动画。

谨记以上，sortedCB实现了SortedList.Callback:
    private SortedList.Callback<String> sortcallback=new SortedList.Callback<String>(){
        @Override
        public int compare(String o1,String o2){
           return o1.compareTo(o2);
      }

      @Override
      public boolean areContentsTheSame(String oldItem,String new Item){
        return(areItemsTheSame(oldItem,new Item));
      }

      @Override
      public void onInserted(int position,int count){
        adapter.notifyItemRangeInserted(position,count);
      }

      @Override
      public void onRemoved(int position,int count){
        adapter.notifyItemRangeRemoved(position,count);
      }

      @Override
      public void onMoved(int fromPosition,int toPosition){
        adapter.notifyItemMoved(fromPosition,toPosition);
      }

      @Override
      public void onChanged(int position,int count){
        adapter.notifyItemRangeChanged(position,count);
      }

    }

第一个是你标准的compare()比较方法，你可能在一个Comparator上实现。从某个排序观点来说如果两个模型对象相等就应该返回0，负数表示第一个参数在第二个参数之前，
正数表示第一个参数在第二个参数之后。

然后就有两个命名相似的方法作为equals()的替代在其中你会有个Comparator的是：areContentsTheSame()和areItemsTheSame()。

在两个传入的值代表同一个实际逻辑项的时候，areItemsTheSame()应该返回true。在SortedFragment这个例子中，仅仅是字符串相等不相等。但是对于更为复杂的数据模型，
你应该比较主键或其他类型的不变的标识。

在项的视觉表现看起来一样的时候，areContentsTheSame（）应该返回ture,因为这个可以用于RecyclerView的优化。

举个例子，假定购物车fragment想要使用SortedList。再假定你添加三袋洗衣粉到购物车中，在RecyclerView中是用三行而不是在一行中标上数量3。
在此例中：
* compare()返回一个值来指示这些物品的排序规则，可能是基于项的标题
* areItemsTheSame（）会返回false，因为这三个是逻辑上不同的行
* areContentsTheSame()会返回true，因为虽然这三个是独立的，它们的内容看起来是一样的。

在很多情形下，areContentsTheSame（）仅仅调用areItemsTheSame()就可以了，在不同项可能有不同的视觉表现这个假定下。这个就是这个样例中所做的，areItemsTheSame()
换而使用compare（）来看项之间是否相同。

最后，四个on...（）方法仅仅转发到它们的RecyclerView.Adapter的对应部分，所以对SortedList的改变是对应部分RecyclerView内容改变了。

注意到，SortedListAdapterCallback把RecyclerView.Adapter作为构造器参数并为你处理on...()方法。但是，因为我们想要保存在配置改变的时候
保持SortedList,并且因为SortedList不允许我们改变SortedList.Callback对象，我们不能毫不犹豫地在配置更改之后把SortedList切换到新的fragment和新的adapter。

####AsyncTask
除了现在单词是添加到SortedList的之外，AddStringTask和原本Async样例中的一样，通过SortedList的回调会更新RecyclerView:
    
    private class AddStringTask extends AsyncTask<Void,String,Void>{
      @Override
      protected Void doInBackground(Void... unused){
        for(String item:items){
          if(isCancelled()){
            break;

            publishProgress(item);
            SystemClock.sleep(400);
          }
          return(null);
        }
        
        @Override
        protected void onProgerssUpdate(String...item){
          if(!isCancelled()){
            model.add(item[0]);
          }
        }

        @Override
        protected void onPostExecute(void unused){
          Toast.makeText(getActivity(),R.string.done,Toast.LENGTH_SHORT).show();
          task=null;
        }

      }
    }


####结果
如果你运行这个样例的话，你会看到每400ms都会有单词添加到列表中。但是，在原本的基于ListView的样例中，并且ListView空间
填满之后你是看不到新的行出现的。在此例中，latin单词由SortedList进行排序，并且你会在它们添加的时候动画进入到合适的位置。
最后，你会得到一个基于CardView的RecyclerView实现，除了单词是被排序过的：

![wizard](/the_busy_coder's_guide/img/figure_448.png)

