###观察者和更新绑定

正如目前在这章看到的，你访问的变量，以及域或方法调用结果，能够填充视图属性。这是有趣的，但是它可能还是没有让你接受数据绑定。虽然有一些较小的代码维护的优点，但是简直不值得。

数据绑定真正闪耀的地方是当你访问的变量，以及域或方法调用结果是可观察对象的时候（例如实现android.databinding.Observable的那个）。然后，表达式不仅在布局资源inflated的时候更新你的视图属性，也会在数据改变的时候更新。如果你有可观察模型，仅仅更新这些模型对象就会自动传递这些改变到任何存在的在观察这些模型的视图上。

例如，假定你在写任务管理清单的时候。用户可以点击CheckBox控件来指示这个特定任务完成了，在这个时候你想要处了更新展现任务的模型对象之外，还要改变整个任务它的RecyclerView行中的绘制。因为CheckBox是相同行的一部分，是和行的模型绑定的，在相同的onClickListener中同时处理UI更新和模型更新可能是容易的。但是，如果你想要在模型改变已经被存储到数据库或网络的时候才更新绘制呢？现在，在onClickListener返回的任意毫秒数过后，你需要更新RecyclerView的某些行...如果正好有行指向这个模型对象。最终，用户可能滑动，或者甚至彻底地离开这个RecyclerView，在此情形中原本的行不应该被改变。

明显的折衷是定义你的模型对象让它使用观察者。稍微不明显的折衷是重新组织你的代码让其有持久模型对象，在这个地方我们进行网络服务服务调用并更新这些模型对象，而不是用新建的实例来取代这些模型对象。后一种方法总的来说打破了数据绑定，并且它在尝试从这些模型更新你的UI的时候就是个大得多的问题了。

####可观察基本类型
整个模型对象本身不需要时可观察的。不管你的绑定表达式使用了什么，就数据而言，必须是可观察的。它可以是单独的域，如果你想要把这些域作为可观察对象发布，比如让它们变成public final。

一个使域称为可观察的的简单办法是，如果域是一个基本类型值(例如，int),把域替换成对应的可观察类(例如..ObservableInt):

	public final ObservableInt score=new ObservableInt();

你的代码可以在可观察的对象之上使用get()和set()方法...基本类型自己会进行基本类型值的get和set封装。调用set()也会通知所有注册的观察者，数据已经改变了，数据绑定系统会使用这个来查明更新你的UI需要什么。

虽然，这可能听起来有点笨拙，但是Java开发这已经在其它地方使用过这个模式了。一个常见的例子就是Atomic...类(例如，AtomicInterger),当这个值可能被多个并行线程进行get和set操作的时候，确保对基本类型的修改是原子性的。
####可观察域

对于非基本类型值来说，整个值的改变是一致的，你可以使用通用的ObservableField方法。特别的，字符串类型不是一个基本类型，而且它是不可改变的，所以改变值就意味着用新的字符串对象替换旧的字符串对象。ObservableField给你设置好可观察字符串：

	public final ObservableField<String> title=
	      new ObservableField<String>();

这只在你用新的对象替换整个对象的时候有效。所以，例如把Location包装进一个可观察域的时候只有在你以替换Location来改变location才有用，而不是在已经存在的Location上调用setLatitude()和setLongitude()方法。替换Location彻底地触发可观察域让其告诉观察这个改变。相比之下，可观察域没有方法知道，你调用了封装对象上的一个方法并以一种观察者需要知道的方式改变了它的状态这件事。

####可观察ArrayList和可观察ArrayMap

数据绑定系统搭载有两个集合类型的可观察类。

一个是,ObservableArrayList，它是相当直接的：它让你在列表里添加和移除成员，并且它这些改变通知给观察者。再一次的，如果你的改变了给定列表成员的状态，是没有让其知道的方式的。只有你改变了列表本身才能知道。

另一个就是ObservableArrayMap。Android在API等级19中添加了ArrayMap。功能上来讲，ArrayMap就像一个HashMap，是一个通过键来访问的集合，尽管还有一些附加的通过数字使用容的API，就如你在ArrayList看到的。这个实现，平衡了内存效率和CPU时间。ObservableArrayMap添加了可观察的特性，ArrayMap内容的改变会报告给观察者。

####自定义可观察者

你可以创建你自己的类来实现Observable接口。最有可能的，你会以扩展BaseObservable的方式实现。

一方面，这不需要很复杂。例如，这就有一个来自数据绑定支持库的ObservableBoolean的实现。

大量代码处理的是ObservableBoolean的[Parcelable序列化]()。从BaseObservable的角度看，关键是调用在set()方法中的notifyChange()。这告诉BaseObservable告诉所有的观察者这个被观察中的东西改变了，如果它们是和这个被观察者绑定的，它们应该去做一些事情。通常来说，“做一些事”会是重新评估绑定表达式并且更新视图的属性，例如更新TextView的文本信息，其中绑定表达式被使用在android:text属性中。

但是，创建更为复杂的自定义被观察者就没有在文档中特别说明了，我们[这章之后内容]()查看。

####一个可观察样例

有了前面的基础，让我们看另一个Stack Overflow样例的演绎。

在目前使用的样例中，通过Stack Excahnge API发布的问题的值有很多。一个是分数，代表净点赞和差评数。在我们之前使用过的问题属性中，唯一有机会变动的是标题，并且者不是经常发生的。另一方面，分数更有可能很快就变了。

所以，[DataBinding/Scored](https://github.com/jinyulei0710/cw-omnibus/tree/master/DataBinding/Scored)样例项目在DataBinding/Static样例项目的基础上添加了分数属性的支持。它也使标题和分数成为可观察的并且在action bar上添加了一个更新的项。点击这个项会更新加载进这个应用的问题的数据，任何对标题和分数的改变会直接反映出来，不用写更新模型的额外代码。
当然，这个样例写的时候是脑子里是没有数据绑定的。尽管之前的样例添加数据绑定的时候没有明显改变应用，但是这次我们肢解我们的代码来获取我们想要的。

####之前样例的限制

我们必须围绕的问题是我们数据模型的类型。

之前的这个样例的版本会通过Retrofit请求模型对象然后把它们拍进一个适配器然后展示在ListView中。从那之前，模型是静态的－没有添加新问题，修改已存在问题的代码。

但是，Retrofit是设计成在每个对网页服务调用接口上创建新的模型对象的。所以，如果一旦我们调用获取最新的问题，然后作出获取更新这些问题的调用，那么我们就以两个独立的模型对象而告终。

如果我们不尝试使用数据绑定，我们可以用一个野蛮的方式，仅仅用新的模型集合替换适配器里面的内容。你会有用，虽然对用于体验有一些影响。(例如，可能列表会滚动到顶端)。

但是，使用数据绑定之后，我们事实上尝试把我们原本的数据模型和我们的视图更加紧密的结合。这意味着当我们从Retrofit获取一系列模型对象的时候，我们不能直接使用它们。作为替代，我们不得不把它们当成一个注入我们原本模型对象的数据源。通过观察着机制，我们可以更新原本的模型而不用担心ListView的行,因为数据绑定会为我们照管。但是着就意味着我们需要有一个“有魔力”的模型集对象来展现绑定数据，并和所有展现数据更新的数据区分开来。

#####Questions对比Items

我们可以通过给予Item从另一个Item更新它的状态的能力来解决这个问题。我们原本最近问题的
的请求回创建Item对象的集合，这会是我们的“持久模型”，它将和我们的UI绑定。之后的更新创建的新Item对回仅仅更新原本持久Item对象的内容，而不是替换这些对象。

但是现在遇到了另一个问题：数据绑定系统的可观察对象需求可能和别处背道而驰。

在这个例子的情形中，在Retrofit接受由服务器返回的Json响应之后，Item是由Gson填充的。Gson不知道关于ObservableField，ObservableInt的任何事情。主要有两个处理这个问题的方法：

1.使用Gson的类型系统尝试教导Gson如何取得JSON属性并更新对应的ObservableField和ObservableInt等模型中的域。极有可能的，这是长期使用的方向，尽管可想到JSON和可观察对象元素有着不相容的不同之处。

2.独立的“模型对象”。一个代表网络服务的调用结果（由Gson填充的）,另一个代表持久化模型(并且由可观察属性)

这个样例的修改版采用了第二种方法。有了一个新的模型类，Question，它塑造出了一个Stack Overflow的问题。我们的数据绑定回应用到Question上。Item仍旧存在，但是它代表了从Stack Exchange网页返回的响应。

#####保存得分(以及ID)

处了处理Question和Item的二重性之外，我们需要多保存两个来自网页服务响应的JSON属性。一个是，之前提及到的分数。另一个是quesiton_id,问题的唯一标识。我们需要这个ID是为了当我们取回更新我们模型的时候，能够使用来自新的Item的数据更新已存在问题。

容易的部分是从Retrofit和Gson获取新的数据。我们仅仅需要在Item中添加用于score和问题ID的域：

	package com.commonsware.android.databind.basic;
	
	import com.google.gson.annotations.SerializedName;
	
	public class Item{
	   String title;
	   Owner owner;
	   String link;
	   int score;
	   @SerializedName("question_id")String 
	}

就问题ID而言，JSON属性是question_id。在Java中，我们会换而使用id。使用Gson的
@SerializedName注解来教导Gson用question_id属性填充到id这个域。

我们现在也有了个会是我们观察对象的Question类，持久的数据模型:

	package com.commonsware.android.databind.basic;
	
	import android.databinding.ObservableField;
	import android.databinding.ObservableInt;
	
	public class Question {
	     public final ObservableField<String> title=
	      new ObservableField<String>();
	      public final Owner owner;
	      public final String link;
	      public final ObservableInt score=new ObservableInt();
	      public final String id;
	      
	      Question(Item item){
	          updateFromItem(item);
	          owner=item.owner;
	          link=item.link;
	          id=item.id;
	      }
	      
	       void updateFromItem(Item item){
	          title.set(item.title);
	          score.set(item.score);
	      }
	}
	
它持有和Item一样的五个值，除了现在title和score现在是分别是通过Observable和ObservableInt创建的，是可观察的。onwer,link和id值应该是不可改变的，反正我们没有在它们上进行绑定，所以保持为普通的域就好了。

Question有一个构造器和一个updateFromItem()方法都是从Item把数据复制到Question中去。updateFromItem()处理了两个可观察域，我们会在最终获取问题更新的时候使用这个方法。	
QuestionsFragment现在有了一个更为恰当的名字，因为现在会用它来展示问题对象的列表。此外，这需要对QuestionsAdapter进行修改，从处理Item对象换成处理Question对象：

		class QuestionsAdapter extends ArrayAdapter<Question>{
		   QuestionsAdapter<List<Question> items){
		     super(getActivity(),R.layout.row,R.id.title,items);
		   }
		   
		   @Override
		   public View getView(int position,View convertView,ViewGroup parent){
		       RowBinding rowBinding=
		         DataBindingUtil.getBinding(convertView);
		       
		      if(rowBinding==null){
		         rowBinding=
		           RowBinding.inflate(getActivity().getLayoutInflater(),
		           parent,false);
		      }  
		      
		      Question question=getItem(position);
		      ImageView icon=rowBinding.icon;
		      
		      rowBinding.setQuestion(question);
		      
		      Picasso.with(getActivity()).load(question.owner.profileImage)
		          .fit().centerCrop()
		          .placeholder(R.drawable.owner_placeholder)
		          .error(R.drawable.owner_error).into(icon);
		          
		       return(rowBinding.getRoot());
		     
		   }
		   
		}

相类似的，row.xml中的<variable>现在需要是一个Question了:

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">

 	 <data>

    <import type="android.text.Html"/>

    <variable
      name="question"
      type="com.commonsware.android.databind.basic.Question"/>
  	</data>

 	 <LinearLayout
  	  android:layout_width="match_parent"
   	  android:layout_height="wrap_content"
      android:orientation="horizontal">

    <ImageView
      android:id="@+id/icon"
      android:layout_width="@dimen/icon"
      android:layout_height="@dimen/icon"
      android:layout_gravity="center_vertical"
      android:contentDescription="@string/icon"
      android:padding="8dip"/>

    <TextView
      android:id="@+id/title"
      android:layout_width="0dp"
      android:layout_height="wrap_content"
      android:layout_gravity="left|center_vertical"
      android:layout_weight="1"
      android:text="@{Html.fromHtml(question.title)}"
      android:textSize="20sp"/>

    <TextView
      android:id="@+id/score"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_vertical"
      android:layout_marginLeft="8dp"
      android:layout_marginRight="8dp"
      android:text="@{Integer.toString(question.score)}"
      android:textSize="40sp"
      android:textStyle="bold"/>

    </LinearLayout>
    </layout>

你会注意到用于得分TextView的绑定表达式是@{Integer.toString(question.score)｝。这是因为
Question上score域是一个整型，并且默认情况下，绑定系统会把它当作是字符串资源的一个引用。我们不得不把得分转化成一个字符串从而得到我们想要的结果。我们会在[这章后面]()看到更多。


#####刷新数据

当然，QuestionsAdapter只有在我们有	Question对象的时候才能适应于Question对象。

QuestionsFragment现在持有了两个Question对象的集合:一个为了从网页服务API获取它们的ArrayList和一个通过给定ID找出Question对象的HashMap。

我们可以换而使用单个ArrayMap或SimpleArrayMap，因为这个结构支持下标和键两种访问方式。但是，我们不得不创建自己的BaseAdapter了，因为ArrayAdapter不知道如何使用ArrayMap或SimpleArrayMap。在此情形中，对于仅仅一个25个问题的组，使用效率不高的两个集合的方法较为简单。

我们在我们StackOverflowInterface接口上的调用仍旧会返回Item对象的集合。在onCreateView()中，我们调用了questions(),我们安排这些Item对象用来创建对应的Question组对象：

	 @Override
  	public View onCreateView(LayoutInflater inflater,
                           ViewGroup container,
                           Bundle savedInstanceState) {
    View result=
        super.onCreateView(inflater, container,
          savedInstanceState);
    Call<SOQuestions> call=so.questions("android");
    call.enqueue(new Callback<SOQuestions>() {
      @Override
      public void onResponse(Call<SOQuestions> call, Response<SOQuestions> response) {
        for (Item item : response.body().items) {
          Question question=new Question(item);

          questions.add(question);
          questionMap.put(question.id, question);
        }

        setListAdapter(new QuestionsAdapter(questions));
      }

      @Override
      public void onFailure(Call<SOQuestions> call, Throwable t) {

      }
    });

     return(result);
    }
    
这就足够让我们的应用再次跑起来了，应用里展示了问题标题，提问人的头像和得分。

![figure_638](/figure_638)       

但是，我们想要允许用户刷新这些问题的数据，所以我们通过数据绑定系统实时看到得分的更新。这需要一个对StackExchangeAPI的不同调用。它仍然是／2.1/quetions,但是现在我们有了附加的路径片段，取的是分号隔开的问题ID。所以，为这个我们在StackOverflowInterface接口上添加了一个新的@GET方法：
		
		package com.commonsware.android.databind.basic;


   		import retrofit2.Call;
		import retrofit2.http.GET;
		import retrofit2.http.Path;
		import retrofit2.http.Query;

		public interface StackOverflowInterface {
 		 @GET("/2.1/questions?order=desc&sort=creation&site=stackoverflow")
 			 Call<SOQuestions> questions(@Query("tagged") String tags);

  		@GET("/2.1/questions/{ids}?site=stackoverflow")
  			Call<SOQuestions> update(@Path("ids") String questionIds);
			}

注意在首个参数上使用的@Path("ids")对应着@GET注解中表现的{ids}占位符。@Path("id")意味着以下参数可以以路径片段的形式注入到URL中，{ids}指示具体参数的值应该去哪。注意，尽管如此，它是一个字符串，不是一个字符串数组或列表。那是因为我们没有办法教导Retrofit把字符串集合拼接到单个路径片段。

此外，这个样例现在有了一个menu资源目录，目录里面有一个acitons.xml资源，其中定义了单个“刷新”菜单按钮。QuesitonsFragment选择把它加到aciton bar,并且在onCreateOptionsMenu中应用了actions菜单资源。在onOptionsItemSelected()中，如果用户选择了我们的刷新按钮菜单，我们就调用一个私有的updateQuestions()方法。这个方法需要在StackOverflowInterface上使用新的update()方法：

	private void updateQuestions() {
    ArrayList<String> idList = new ArrayList<String>();

    for (Question question : questions) {
      idList.add(question.id);
    }

    String ids = TextUtils.join(";", idList);

    Call<SOQuestions> call = so.update(ids);
    call.enqueue(new Callback<SOQuestions>() {
      @Override
      public void onResponse(Call<SOQuestions> call, Response<SOQuestions> response) {
        for (Item item : response.body().items) {
          Question question = questionMap.get(item.id);

          if (question != null) {
            question.updateFromItem(item);
          }
        }
      }

      @Override
      public void onFailure(Call<SOQuestions> call, Throwable t) {

      }
    });

 	 }

我们收集了所有的问题ID,然后使用TextUtils.join()来给予我们所有问题ID用分号拼接单个字符串。这个字符串里的id，会依次传递给update()。对于每个返回的Item,我们会找到HashMap中对应的问题并用来自Item的最新数据更新它。

我们没有去碰我们的UI。

但是，如果你运行应用，从列表中选择一个好问题，为问题点赞，然后刷新列表，你会看到新的得分在刷新之后立即出现了。数据绑定系统为我们进行了这个处理，而不用我们做额外的手工措施。



