###其它需要注意的属性特征

还有很多你可以在数据绑定中应用的花哨功能。

####通过绑定类获取视图

样例应用已经从RowBinding中取回了用于行的ImageView控件。布局文件中所有的视图都有一个android:id值都对应着一个绑定生成类中的域。所以，像Picasso这样的情形，我们不能使用数据绑定来填充Imageview并必须采取传统的绑定它到适配器的逻辑，我们不需要自己做findViewById()调用。取而代之的，我们仅仅访问绑定类中的域就可以了。

####操作绑定中的变量

我们已经看过生成绑定类使用一个setter方法来把一个对象绑定到一个布局上。在样例应用中，我们已经调用了setItem()或setQuestion()来提供模型对象在绑定表达式中使用。如果需要的话，还有对应的getter方法（getItem(),getQuestion())来取回最后设置的值。

####视图,设置方法，和绑定

我们已经看过在android:text中使用绑定表达式，来为TextView设置文本。

实际上发生的事：

* 绑定系统评估表达式。着不仅给予我们要绑定的值也决定了值的数据类型。
* 数据绑定系统查找叫做set...（）的设置方法，其中...部分是基于属性名的（减去所有的命名空间），其中数据类型对应着表达式结果的数据类型。所以，在绑定表达式为android:text属性生成一个字符串的时候，数据绑定系统会在空间上查找setText(String)方法，在我们的情况中就是一个TextView。如果绑定表达式返回的是int,作为替代，数据绑定系统会查找setText(int)方法。在TextView的情形中，这个就存在，并且它预期int是一个字符串资源。这就是为什么，在Scored样例应用中，我们需要把整型转化为字符串。

当然，这仅仅是个简单的情形。

#####合成属性

数据绑定系统把属性名映射到setters。但是，如果你使用的属性名事实上不存在呢？

所有数据绑定系统做的事情是使用属性名来尝试找到一个相关的setter方法。属性名是不是LayoutInfalter支持的XML的结构是无关重要的。

这意味着你可以使用任意属性来映射到一个setter方法。

例如,ViewPager除了它从View或ViewGroup继承来的属性之外，没有它自己的XML属性。但是，你是欢迎使用像app:currentItem或app:pageMargin在你数据绑定增强的布局资源的（其中app指向一个你的自定义命名空间)。LayoutInfalter会分析它们，但是ViewPager会忽略它们。但是，数据绑定系统会高兴地让你绑定值到它们上，分别触发对setCurrentItem()和setPageMargin()的调用。

因此，不要感觉你是被限定只能使用这些由LayoutInflter和控件官方支持的属性。如果数据绑定系统可以找到一个setter,你就能使用它。

但是，这些合成属性有一个关键的限制，值必须是一个绑定表达式。这是真的，即使你没有真正地很好地进行表达式评估。

例如，这个就没有用:

	<ImageView
	 android:id="@+id/icon"
	 android:layout_width="@dimen/icon"
	 android:layout_height="@dimen/icon"
	 android:layout_gravity="center_vertical"
	 android:contentDescription="@string/icon"
	 android:padding="8dip"
	 app:error="@drawable/owner_error"
	 app:imageUrl="@{question.owner.profileImage}"
	 app:placeholder="@drawable/owner_placeholder"/>
	 
这里，我们有三个合成属性，app:error,app:imageUrl,和app:placeholder。只有app:imageUrl是使用了一个绑定表达式，并且它的使用是说的过去的，因为我们是从一个变量（question）拉取数据的。其它两个指向图片。按照理想的话，这个会有用。实际上，它并没有用，因为绑定系统忽略了这个属性，并且Android会投诉属性没有识别。

但是，这个就有用了:

	<ImageView
 		android:id="@+id/icon"
 		android:layout_width="@dimen/icon"
 		android:layout_height="@dimen/icon"
 		android:layout_gravity="center_vertical"
		 android:contentDescription="@string/icon"
 		android:padding="8dip"
 		app:error="@{@drawable/owner_error}"
 		app:imageUrl="@{question.owner.profileImage}"
 		app:placeholder="@{@drawable/owner_placeholder}"/>
 
现在,app:error和app:placeholer使用了绑定表达式...这个表达式仅仅返回了一个drawable资源引用。如果下面满足下面条件之一，这就会有用：
	 
1.在ImageView上存在给这些属性的setter方法(例如,setError()),在这个情形中，并没有,或者

2.我们使用其它的技术告诉数据绑定系统这些属性路由到了其它地方，就如在接下来两节会看到的

####使用不同的方法

当然，找到一个setter方法可能是个挑战。经常，属性名和setter名汇遵循描述约束（android:foo映射setFoo()）。尽管如此，时不时地属性名和setter名会不同。

例如，View有一个android:fadeScroolbars属性用来决定当控件不滚动的时候是否应该自动褪去scrobars。但是，相关的setter方法不是setFadeScroolbars(),而是setScroolbarFadingEnabled()。默认情况，理论上来说，数据绑定系统不会找到为android:fadeScroolBars找到合适的setter方法。

实际上，文档表明Google已经修复了所有来自Android 框架类的标准属性。但是，可能有缺口，特别在Android支持提供的类中，更不用说第三方控件了。

为了克服attribute/setter对不匹配的问题，你可以教导数据绑定系统如何找到给属性的setter方法。
为了完成这个工作，你是应当能够定义一个类级别的@BindingMethod注解，其中包含一个或多个@BindingMethod注解，它相应地把一个属性映射到一个setter方法名：

	@BindingMethod(
	   @BindingMethod(type="android.view.View",
	                   attribute="android:fadeScrollBars"
	                   method="setScroolbarFadingEnabled"),
	   })

####绑定适配器，和Picasso情景

有时候，即使这么做也是不够的。可能setter方法取了额外的参数，尽管在你的情形中它们可以简单地进行硬编码或从控件其它地方拉取。可能“setter方法”不是真正地在设置属性，但是安排做一些与属性相关的工作。

例如，目前为止，我们还没有能够在ImageView上使用数据绑定。虽然图片的URL是与android:src属性相关的，但是android:src并不接收一个URL，并且反正是要使用Picasso来异步取回图片的。因此我们被迫接受在getView()以老式的方法来配置ImageView，取回ImageView然后告诉Picasso如何填充它。

但是，绑定系统也能通过定义一个自定义@BindingAdapter来处理这个问题。

让我们看看[DataBind/Picasso](https://github.com/jinyulei0710/cw-omnibus/tree/master/DataBinding/Picasso)样例项目。这个项目是基于之前的Scored样例的，但是现在使用了
数据绑定系统来更新ImageView。

稍早出现的ImageView XML出现在了我们修改过的row.xml布局资源中:

    <?xml version="1.0" encoding="utf-8"?>
	<layout
 	 xmlns:android="http://schemas.android.com/apk/res/android"
 	 xmlns:app="http://schemas.android.com/apk/res-auto">

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
      android:padding="8dip"
      app:error="@{@drawable/owner_error}"
      app:imageUrl="@{question.owner.profileImage}"
      app:placeholder="@{@drawable/owner_placeholder}"/>

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

这里我们有三个合成属性：属性不是真正ImageView的一部分，但是我们我们借助了数据绑定系统。

为了这个能奏效，数据绑定系统必须知道对这三个值做什么。ImageView缺少这些的setter方法(),
所以在没有其它东西时，数据绑定系统会触发编译错误，控诉它不知道要对我们布局中的值做什么。

为了这个能奏效，我们需要在某处放一个带着@BindingAdapter注解静态的方法。在这个情形中，我们在QuestionsFragment中定义了它：

      @bindingAdapter({"app:imageUrl","app:placeholder","app:error"})
      public static void bindImageView(ImageView iv,
                                    String url,
                                    Drawable placeholder,
                                    Drawable error){
         Picasso.with(iv.getContext())
                .load(url)
                .fit()
                .centerCrop()
                .placeholder(placeholder)
                .error(error)
                .into(iv);                              
       }

方法名并不重要，所以把它命名成能提醒你它的职责就行。它需要返回void,并取得如下参数:

* 合成属性会出现在的视图类型(在此例中，ImageView)
* 这些属性的值，以它们出现在@BindingAdapter注解字符串列表中的顺序

在我们的情形中，app:placeholder和app:error是解析至Drawable资源的，而app:imageUrl
是解析至字符串的。

这个声明教导数据绑定系统在它找到一个带着合成属性列表的指派类型视图的任意时间调用这个方法，而不用尝试找到这些属性的setter方法。因为当我们布局文件中的<ImageView>元素满足这些条件的时候，bindImageView()方法就会调用。

在这个方法中，就是我们的职责去做无论我们需要做的事情去摄入这些合成属性值然后应用它们的结构到提供的视图上。但是，之前，drawable的值（placeholder和error）在Java中是硬编码的。现在，它们处于布局XML文件中，这个就有了更多的灵活性，特别是当我们为不同配置使用不同布局资源的时候。

这就意味着我们可以丢弃来自getView()的手工绑定代码，只剩下我们ArrayAdapter到RowBinding的连接操作。

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
      RowBinding rowBinding=
        DataBindingUtil.getBinding(convertView);

      if (rowBinding==null) {
        rowBinding=
          RowBinding.inflate(getActivity().getLayoutInflater(),
            parent, false);
      }

      rowBinding.setQuestion(getItem(position));

      return(rowBinding.getRoot());
    }

注意，尽管如此，为了能够让这个样例奏效，我们需要做另一处改变。app:imageUrl引用了
Owner类中的profileImage域。以前，这是包私有的，这就意味着数据绑定生成代码不能访问它。
取而代之的，我们必须使它成为公共的：
    
      package com.commonsware.android.databind.basic;

     import com.google.gson.annotations.SerializedName;

      public class Owner {
        public @SerializedName("profile_image") String profileImage;
       }

作为一个附加特性，绑定适配器不仅能接受属性的新值，而且也能接受旧的（也就是说，在前一个绑定中使用过的）。为了使这个能奏效，你把除了视图之外的参数所有参数加倍。首先是旧的值，然后是新的值。如果我们想要在这节展示这个样例中使用的话，我们总共需要七个参数：

	@BindingAdapter({"app:imageUrl","app:placeholder","app:error"}){
	      public static void bindImageView(ImageView iv,
	                    String oldUrl,
	                    Drawable oldPlaceholder,
	                    Drawable oldError,
	                    String newUrl,
	                    Drawable newPlaceholder,
	                    Drawable newError){
	    //do good sutff here                
	  }

   
####事件处理
目前为止，我们关注的是使用绑定表达式返回填充控件的数据，具体来说，就是配置控件的样子。

但是如何配置控件的行为呢？

结果是目前涉及到的绑定简单数据同时也可以在控件上绑定事件监听者。主要是因为，ViewBindingAdapter类，是数据绑定类库内部实现的一部分。这个类声明了一打的@BindingMethod和@BindingAdapter注解，其中很多是为了允许你在合成属性上使用绑定表达式来指定事件监听者而设计的。

使用这个是不是一个好主意另说。一方面，它减少了连接控件的样板化Java代码的数量。另一方面，有些然可能担心控制层和表现层的分割线会变得模糊不清。

这点数据绑定系统中的特定功能也遭受了有限文档的问题。心中想着那个,[DataBind/RecyclerView](https://github.com/jinyulei0710/cw-omnibus/tree/master/DataBinding/RecyclerView)样例项目展示了这是如何工作的，以及如何使用数据绑定系统来填充RecyclerView取代AdapterView。

#####转变到RecyclerView/CardView UI

首先，与数据绑定无关的，我们需要把应用迁移到使用RecyclerView。与此同时，我们也可以添加CardView的支持，来让竖直滚动的列表的单个成分看起来是带有圆角和落影的卡片。

为此，我们在我们的build.gradle依赖清单中添加了recyclerview-v7和cardview-v7:

	dependencies{
	    compile 'org.greenrobot:eventbus:3.0.0'
        compile 'com.squareup.picasso:picasso:2.5.2'
        compile 'com.squareup.retrofit2:retrofit:2.0.0'
        compile 'com.squareup.retrofit2:converter-gson:2.0.0'
        compile 'com.android.support:recyclerview-v7:23.2.1'
        compile 'com.android.support:cardview-v7:23.2.1'
	}

我们之前的样例使用的是ListFragment。我们没有由RecyclerView-v7类库提供的RecyclerViewFragment。但是我们可以有我们自己的，从RecyclerView样例项目复制而来的：

     
	package com.commonsware.android.databind.basic;

	import android.app.Fragment;
	import android.os.Bundle;
	import android.support.v7.widget.RecyclerView;
	import android.view.LayoutInflater;
	import android.view.View;
	import android.view.ViewGroup;

	public class RecyclerViewFragment extends Fragment {
	  @Override
 	 public View onCreateView(LayoutInflater inflater, ViewGroup container,
	                           Bundle savedInstanceState) {
 	   RecyclerView rv=new RecyclerView(getActivity());

  	  rv.setHasFixedSize(true);

   	 return(rv);
 	 }

  	public void setAdapter(RecyclerView.Adapter adapter) {
   	 getRecyclerView().setAdapter(adapter);
 	 }

 	 public RecyclerView.Adapter getAdapter() {
  	  return(getRecyclerView().getAdapter());
 	 }

  	public void setLayoutManager(RecyclerView.LayoutManager mgr) {
 	   getRecyclerView().setLayoutManager(mgr);
 	 }

  	public RecyclerView getRecyclerView() {
  	  return((RecyclerView)getView());
 	 }
	}
	
所有这个类做的事自己管理一个RecyclerView，包括允许我们自己来操作适配器和布局管理器。

修改过的QuestionsFragment现在继承了RecyclerViewFragment。我们在onViewCreate()中配置了RecyclerView，用的大部分事之前的代码，除了我们也需要调用setLayoutManager()来指示我们想要如何对items进行摆放－在这个情形中，选择的事竖直滚动的列表：

	@Override
  	  public void onViewCreated(View view,
         	                     Bundle savedInstanceState) {
        	super.onViewCreated(view, savedInstanceState);

       	 setLayoutManager(new LinearLayoutManager(getActivity()));


        	Call<SOQuestions> call = so.questions("android");
        	call.enqueue(new Callback<SOQuestions>() {
          	  @Override
          	  public void onResponse(Call<SOQuestions> call, 	Response<SOQuestions> response) {
              	  for (Item item : response.body().items) {
              	      Question question = new Question(item);

                   	 questions.add(question);
                  	  questionMap.put(question.id, question);
               	 }

                setAdapter(new QuestionsAdapter(questions));
           	 }

            	@Override
            	public void onFailure(Call<SOQuestions> call, Throwable t) {

           	 }
        	});

    	}	


我们的QuestionsAdapter也需要改变成一个RecyclerView.Adapter,不能再是ArrayAdapter了：

 	class QuestionsAdapter
      	      extends RecyclerView.Adapter<QuestionController> {
        	private final ArrayList<Question> questions;

        	QuestionsAdapter(ArrayList<Question> questions) {
       	     this.questions = questions;
       	 }

        	@Override
        	public QuestionController onCreateViewHolder(ViewGroup parent,
                                                     int viewType) {
           	 RowBinding rowBinding =
             	       RowBinding.inflate(getActivity().getLayoutInflater(),
                 	           parent, false);

            	return (new QuestionController(rowBinding, this));
       	 }

       	 @Override
       	 public void onBindViewHolder(QuestionController holder,
        	                             int position) {
        	    holder.bindModel(getItem(position));
       	 }
	
       	 @Override
       	 public int getItemCount() {
        	    return (questions.size());
        	}

        	Question getItem(int position) {
         	   return (questions.get(position));
       	 }
    	}

我们取回了构造器中问题的清单并存放起来以备之后使用。getItemCount()和getItem()仅仅是访问量了问题的清单。数据绑定发生在onCreateViewHolder()中，其中我们创建了RowBinding并使用它来设置了
QuestionController。QuestionController是一个RecyclerView.ViewHolder的子类,并作为我们列表中用于行的本地控制器－我们很快就会看到QuestionController更多详尽的内容。onBindViewHolder()仅仅是告诉QuestionController来绑定到提供的Question模型对象上。

RecyclerView.ViewHolder需要行的根视图被提供给它的构造器。所以在QuestionController构造器中，我们调用了getRoot()来从RowBinding获取视图并把它提供出来，同时把RowBinding和QuestionsAdapter存放在域中：

	public QuestionController(RowBinding rowBinding,
        	                    QuestionsFragment.QuestionsAdapter adapter) {
   	 super(rowBinding.getRoot());

  	  this.rowBinding=rowBinding;
    	this.adapter=adapter;
	  }

并且，在bindModel()中，我们使用了RowBinding来绑定我们的问题，所以绑定表达式会把标题，得分等等拉进我们的视图：

	void bindModel(Question question) {
    	rowBinding.setQuestion(question);
    	rowBinding.setController(this);
 	 }

你会注意到我们也在RowBinding上调用了setController()方法。这是因为我们事件绑定工作的支撑，你会在之后两节中看到。

#####发布事件监听者

前面提到的ViewBindingAdapter类为android:onClick和android:onTouch创建了绑定方法，各自取的是onClickListner和onTouchListener。为了绑定监听者，我们需要的是一个能够发布监听者的对象，所以我们可以通过绑定表达式把对象注入到视图上。

在我们的情形中，QuestionController就是发布监听者的那个。

OnclickListener是由controller本身仅仅实现onClickListener接口而进行处理的，onClick()中是我们的事件总线逻辑：

	 	@Override
 	 public void onClick(View v) {
    	Question question=adapter.getItem(getAdapterPosition());

    	EventBus.getDefault().post(new QuestionClickedEvent(question));
 	 }

我们可以以相同的方式去处理onTouchListener，但是这个样例阐释了另一种方法：使用了一个公共的静态域：

	public static final View.OnTouchListener ON_TOUCH=
   	 new View.OnTouchListener() {
     	 @TargetApi(Build.VERSION_CODES.LOLLIPOP)
     	 @Override
     	 public boolean onTouch(View v, MotionEvent event) {
        	if (Build.VERSION.SDK_INT>=
        	  Build.VERSION_CODES.LOLLIPOP) {
         	 v
          	  .findViewById(R.id.row_content)
            	.getBackground()
            	.setHotspot(event.getX(), event.getY());
        	}

       	 return(false);
      	}
    	};
    	
在这个情形中，我们所需要来处理触摸事件的一切，都是以参数到形式到onTouch()方法中的，这就是为什么一个静态域能有用的原因。

另一个可能就是让一个实例方法返回一个监听者，它在监听者需要来自控制层数据的时候会比较好。

#####应用事件监听者

在这之后，添加监听者到布局资源中就大致上就很平淡了。我们需要一个新的<variable>用于我们的QuestionController,并且我们需要添加android:onClick和android:onTouch属性到LinearLayout上：

	<?xml version="1.0" encoding="utf-8"?>
	<layout
 	 xmlns:android="http://schemas.android.com/apk/res/android"
 	 xmlns:app="http://schemas.android.com/apk/res-auto">

  	<data>

   	 <import type="android.text.Html"/>

    	<variable
     	 name="question"
      	type="com.commonsware.android.databind.basic.Question"/>

    	<variable
     	 name="controller"
      	type="com.commonsware.android.databind.basic.QuestionController"/>
  	</data>

  	 <android.support.v7.widget.CardView
     xmlns:cardview="http://schemas.android.com/apk/res-auto"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:layout_margin="4dp"
     cardview:cardCornerRadius="4dp">

    <LinearLayout
      android:id="@+id/row_content"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:background="?android:attr/selectableItemBackground"
      android:onClick="@{controller}"
      android:onTouch="@{controller.ON_TOUCH}"
      android:orientation="horizontal">

      <ImageView
        android:id="@+id/icon"
        android:layout_width="@dimen/icon"
        android:layout_height="@dimen/icon"
        android:layout_gravity="center_vertical"
        android:contentDescription="@string/icon"
        android:padding="8dip"
        app:error="@{@drawable/owner_error}"
        app:imageUrl="@{question.owner.profileImage}"
        app:placeholder="@{@drawable/owner_placeholder}"/>

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
  </android.support.v7.widget.CardView>
</layout>

android:onClick的绑定表达式仅仅是@{controller},是因为QuestionController实现了
OnClickListener接口。

####类型转换

绑定表达式的结果可以转换成setter,域，或数据绑定系统识别处要使用的绑定适配器。

如果顺利的话，这会奏效。

但是，又可能你会需要改变你的绑定表达式，例如在这章之前提出的情形之中，其中android:text可以接收整型，但是你想要整型以文本的形式展示，而不是对字符串资源的引用。

在其它情形中，可能没有清晰的匹配。Google的文档提出了这么一个例子，这个例子中你的绑定表达式返回了颜色资源的ID,但是setter方法却取了一个Drawable，就如这个情形中视图上的setBackground()方法一样。

解决这个不一致性的一种方法是通过一个@BindingMethod。这会教导数据绑定系统给setter使用不同的方法（例如setBackgroundColor()）。但是，这个总是被特定控件和属性的结合体所使用的。在android:background属性这个特别情形中，有着多个可能的setters:

* setBackgound(Drawable)
* setBackgroundColor(int)（取的是一个真正的颜色，而不是一个色彩资源）
* setBackgroundDrawable(Drawable)（API等级16新增的setBackground(Drawable)）
* setBackgroundResource(int)

你可能不能特定地使用它们其中的某一个。

因此，另一方法是教导数据绑定系统如何将数据从一个类型转换为另一个类型，使用一个@BindingConversion注解的静态方法：

	@BingdingConversion
	public static ColorDrawable colorToDrawable(int color){
	  return new ColorDrawable(color);
	}
 
和绑定适配器一样，方法的名字并不重要。重要的是它取一个整型座位输入并返回了一个ColorDrawable。
数据绑定系统会记住这个并在绑定表达式返回整型和它需要一个ColorDrawable..或一个Drawable的时候使用。

这里，尽管我们开始跑进了Google坚持在到处使用int值而引发的问题。这个colorToDrawable()转换方法取的是一个整型。这个整型可能是一个颜色。它可能是颜色资源ID，或者是一个字符串资源ID,或者是布局资源ID,或者是Stack Overflow问题的得分，又或者是其它数不清的东西。描述的@BindingConversion，因此，可能不会特别有用。

另一种使用@BindingConversion的情形是能够从模型深处获取一些东西而不用把整个模型结构暴露成公共的。例如，[DataBinding/Conversion]()样例项目使用了一个@BindingConversion，通过返回profileImage值的方式来允许Owner转变成一个字符串：

    @BingdingConversion
    public static String ownerToString(Owner owner){
       return(owner.profile);
    }

再一次的，方法名并不重要，重要的是这个转换知道如何处理取得一个Owner并返回一个字符串。

现在，布局中ImageView中app:imageUrl属性可以引用question.owner而不是question.owner.profileImage了:
    
     <ImageView
        android:id="@+id/icon"
        android:layout_width="@dimen/icon"
        android:layout_height="@dimen/icon"
        android:layout_gravity="center_vertical"
        android:contentDescription="@string/icon"
        android:padding="8dip"
        app:error="@{@drawable/owner_error}"
        app:imageUrl="@{question.owner}"
        app:placeholder="@{@drawable/owner_placeholder}"/>
        
        
####自定义绑定类名

正如之前在这章提到的，用于布局资源的绑定类名是默认自动决定的。布局文件名转换成了帕斯卡拼写法，然后附加了Binding（例如，res/layout/foo_bar.xml变成了FooBarBinding）。这个类到了你应用包名的.databind子包之下，而这个包名和在你的<manifest>中定义的package属性一样。

但是，这可能造成Java类名的尴尬。或者，由于某种原因你可能想要把类生成在其它Java包中。你可以使用<data>元素上的classs属性来控制被用于绑定类的真实Java类名。

可以以以下三种形式中的一种出现：

* class="Foo"会把绑定类命名成Foo并会把它放在标准的.databinding子包中
* class=".Foo"会把绑定类命名成Foo并放在你应用基包的下面，而不是在单独的.databinding子包下。
* class=this.is.fully.qualified.Foo"会把绑定类命名成Foo并把它放进指定的Java包中。

####扩展include句法

Android从1.0开始就在布局资源中支持<include>标签了。这个标签取的是一个layout属性，引用的是一个layout资源。所指向的布局资源内容插入到了原本资源的视图层级中。所以，我们有：
	
	<LinearLayout xmlns:android:"http://schemas.android.com/apk/res/android"
	android:orientation="vertical"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
	<include layout="@layout/foo"/>
	<!--other widgets go here-->
	</LinearLayout>

...然后不管foo布局中有什么都会添加到LinearLayout中，在LinearLayout其它控件之前。

使用数据绑定系统，你可以把变量从外部布局传到包含的那个布局中，而不用从Java代码层面不知以某种方式绑定变量到所包含的布局上：

	<layout xmlns:android:"http://schemas.android.com/apk/res/android"
     	   xmlns:bind="http://schemas.android.com/apk/res-auto"
     <data>
          <variable name="foo" type="com/thingy.Foo"/>
      </data>
      
      <LinearLayout xmlns:android="http:schemas.android.com/apk/res/android"
          android:orientation="vertical"
          android:layout_width="match_parent"
          android:layout_height="match_parent">
       <include layout="@layout/foo" bind:bar="@{foo}"       	   	<!--other widget go here-->
       </LinearLayout>
       </layout>

这里，如果foo布局资源有一个变量叫做bar的话，它会由以评估@{foo}绑定表达式的方式填充进去，
那么foo资源就能引用到它自己绑定表达式中的bar了。

####TODO


