###DIY HTTP

在很多情况下，你唯一切实可行访问一些网络服务或者其它基于HTTP协议的资源的时候的方法是自己请求。Google认可的用于HTTP操作的类是传统的java.net包下的类，围绕着HttpUrlConnection。关于这个的材料发布的已经有很多了，因为这么类
在Java中已经存在有相当长一段时间了。这里关注的是展示它如何是在Android情境中工作的。

注意，然而你可能发现使用一些HTTP客户端类库是容易的，它们为你处理了网络访问多个方面的内容，正如会在[稍后这章]()里描述的那样。

####HttpUrlConnection使用样例

这章实现的是StackOverdlow客户应用。应用有着单个activity,activity里面是单个ListFragment。应用会使用StackExchange公用接口来展示StackOverflow上标签为android的问题。这些问题会展示在列表中，并且点击问题时会在用户默认浏览器中出现一个这个问题的网页。

这个应用的所有实现都有相同的核心UI逻辑。区别是，每个处理网络访问的方式不同。在这节中，我们会看下[Internet／HURL](https://github.com/commonsguy/cw-omnibus/tree/master/Internet/HURL)这个样例项目，它使用了HttpUrlConnection来从Stack Exchange 网络服务API获取问题。

####请求权限
应用做网络连接相关的事情时候，你需要拥有INTERNET权限。这就包含你使用WebView的时候－如果你的处理需要网络访问，你就需要INTERNET权限。

因此，我们样例项目的清单中必不可少如下<uses-permission>声明：

	<uses-permission
	  android:name="android.permission.INTERNET"/>

	  
####创建你的数据模型
对Stack Exchange网络服务API进行多种请求的时候会返回JSON。因此，我们需要创建Java类来反映这个JSON结构。特别的，许多样例会使用Google的Gson把从网络服务获取的JSON自动分析成数据模型。

在我们的例子中，我们要使用的是Stack Exchange API中的一个特定端点，被称作/question，它处于路径区分部分之后。这个端点的文档能在[Stack Exchange API文档](https://api.stackexchange.com/docs/questions)中找到。

我们在这节里看下这个端点的URL。

GET请求这个接口之后得到的是一个JSON结构(为了简短，这里展示了单个问题): 

     {
        “items”:[
           {
            "question_id":17196927,
            "create_date":137660594,
            "last_activity_date":1371660594,
            "score":0,
            "answer_cout":0,
            "title":"ksoap failing when in 3G",
            "tags":[
              "android",
              "ksoap",
              "3g"
            ],
            "view_count":2,
            "owner":{
            "user_id":773259,
            "display_name","Spark",
            "reputation":513,
            "user_type":"registered",
            “profile_image”:
        "http://www.gravatar.com/avatar/511b37c313984e624dd76e8cb
        9faa6?d=identicon&r=PG",
             "link":
            "http://stackoverflow.com/users/773259/spark"
            },
            "link":
           "http://stackoverflow.com/questions/17196927/ksoap2-failing-when-in－3g",
           "is_answered":false      
           }
        ],
        "quota_remaining":9991
        "quota_max":10000,
        "has_more":true
     }

注意：一些较长的URLs换行了，但是在真正的JSON中是单行的。

我们获取的是一个JSON对象，我们的问题是在items的名称下发现的。items是一个json对象组成的JSON数组，每个JSON对象用标题和链接代表一个问题。问题的JSON对象还嵌入了一个所有者的JSON对象作为附加信息。

我们没必要需要所有这些信息，实际上，对于这个样例的首个版本来说，所有我们所真正需要的是item数组中的标题和链接。   

关键是，默认情况下，我们Java数据模型中的成员变量必须真正和JSON对象的JSON键值匹配。

所以呢，我们有了一个Item类，代表来着items数组中的一个独立入口的信息：
	package com.commonsware.android.hurl;
	
	public class Item{
	   String title;
	   String link;
	   
	   @Override
	   public String toString(){
	       return(title);
	   }
	}

但是，我们的网页服务并没有直接返回items数组。items是JSON对象对象的键，那是由StackExchange返回的真正JSON。所以，我们需要另一个包含我们需要的来自外部JSON对象的数据成员，这里的名称是SOQuestions（没有更好关于姓名的主意了...）：

	package com.commonsware.android.hurl;
	
	import java.utils.List;
	
	public class SOQuestions{
	  List<Item> items;
	}
有一个是Item组成的列表的items的数据成员告诉GSON我们期待JSON对象被SOQuestions
使用为一个叫做items的JSON数组，数组中的每个元素应该映射到Item对象。

####用于加载的一个线程

我们需要在后台线程去做网络I/O，所以我们没有绑定到主应用线程。为此，样例应用有一个
加载我们问题的LoadThread:

		 package com.commonsware.android.hurl;
		 
		 import android.utils.Log;
		 import java.io.BufferedReader;
		 import java.io.IOException;
		 import java.io.InputStream;
		 import java.io.InputStreamReader;
		 import java.net.HttpURLConnection;
		 import java.net.URL;
		 import com.google.gson.Gson;
		 import de.greenrobot.event.EventBus;
		 
		 class LoadThread extends Thread{
		    static final String SO_URL=
		    "https:/api.stackexchange.com/2.1/questions?"
		    +
		    "order=desc&sort=create&site=stackoverflow&tagged=android";
		    @Override
		    public void run(){
		      try{
		       HttpURLConnection c=(HttpURLConnection)new   URL(SO_URL).openConnection();
		       try{
		       InputStream in=c.getInputStream();
		       BufferedReader in=c.getinputStream();
		       BufferedReader reader=
		           new BufferedReader(new InputStreamReader(in));
		        SOQuestions questions=
		           new Gson().fromJson(reader,SOQuestions.class);
		           
		           reader.close();
		           
		           Event.getDefault().post(new QuestionsLoadedEvent(questions));
		    }
		    catch(IOException e){
		      Log.e(getClass().getSimpleName(),"Exception parsing JSON",e);
		    }finally{
		      c.disconnect();
		    }
		 }
		 catch(Exception e){
		    Log.e(getClass().getSimpleName(),"Exception paring JSON",e);
		 }
		}
	}

LoadThread:

* 通过我们的Stack Exchange API端点创建一个HttpUrlConnection并打开一个连接
* 从HTTP连接创建一个包裹InputStream的一个BufferedReader
* 通过一个Gson实例解析我们从HTTP请求获取的JSON，然后把数据记载进一个我们SOQuestions的一个实例。
* 关闭BufferedReader（以及扩展的InputStream）
* 邮寄一个QuestionsLoaderEvent到greenrobot的EventBus，让某人知道我们问题的存在
* 出错时打印信息到LogCat

QuestionsLoadedEvent是对一个SOQuestions实例的简单包装，作为EventBus的事件类使用.

	 package com.commonsware.android.hurl; 
	 public class QuestionsLoadedEvent｛
	   final SOQuesitons questions;
	   
	   QuestionsLoadedEvent(SOQuestions quesitons){
	     this.questions=questions;
	   }
	 ｝
	
用于问题的碎片

样例应用有一个应该展示这些加载后问题的QuestionsFragment：

	package com.commonsware.android.hurl;
	
	import android.app.ListFragment;
	import android.os.Bundle;
	import android.text.Html;
	import android.text.View;
	import android.view.ViewGroup;
	import android.widget.ArrayAdapter;
	import android.widget.ListView;
	import android.widget.TextView;
	import java.util.List;
	import de.greenrobot.event.EventBus;
	
	public class QuestionsFragment extends ListFragment{
	   @Override
	   public void onCreate(Bundle savedInstanceState){
	      super.onCreate(savedInstanceState);
	   
	      setRetainInstance(true);
	      new LoadThread().start();   
	   }
	   
	   @Override
	   public void onResume(){
	       super.onResume();
	       EventBus.getDefault().register(this);
	   }
	   
	   @Override
	   public void onPause(){
	        EventBus.getDefault().unregister(this);
	        super.onPause();
	   }
	   
	   @Override
	   public void onListItemClick(ListView l,View v,int
	   position,long id){
	      Item item=((ItemsAdapter)getListAdapter()).getItem(position);
	      EventBus.getDefault().post(new QuestionClickedEvent(item));
	   }
	   
	   public void onEventMainThread(QuestionsLoadedEvent event){
	      setListAdapter(new ItemsAdapter(event.quesitons.items));
	   }
	   
	   class ItemsAdapter extends ArrayAdapter<Item>{
	      ItemsAdapter(List<Item> items){
	         super(getActivity(),
	         android.R.layout.simple_list_item_1,items);
	      }
	      
	      @Override
	      public View getView(int position,View convertView,
	      ViewGroup parent){
	          View row=super.getView(position,convertView,parent);
	          TextView title=(TextView)row.findViewById(android.R.id.text1);
	          
	          title.setText(Html.fromHtml(getItem(position).title));
	          
	          return(row);
	      }
	      
	   }
	      
	}
	
在onCreate()中，我们标记这个fragment应该保留并fork出LoadThread。
	
因此，一旦我们有了我们的问题，我们保留的fragment会给我们保留模型数据，并且我们避免在fragment初始化之后配置改变时重复创建LoadThread。
	
在onResume()和onPause()中，我们注册以及注销了EventBus。我们的onEventMainThread（）方法会在QuestionLoadedEvent被LoadThread提出的时候调用，然后我们就有了加载下来问题并填充ListView。我们使用了一个ItemAdapter，它知道如何把简单的ListView行展示问题的标题。ItemsAdapter使用Html.fromHtml()来填充ListView行，不是因为stack overflow用html标签返回标题，而是因为用html实体引用返回标题，并且Html.fromHtml()应该能处理它们中的很多。

并且，在onListItemClick()，我们在用户点击的时候找到与行相关的Item，然后提出一个QuestionClickEvent来让某人知道用户点击了那一行。QuestionClickedEvent类时一个简单包装Item而被EventBus使用的事件类:

	package com.commonsware.android.hurl;
	
	public class QuestionClickEvent{
	   final Item item;
	    
	    QuestionClickedEvent(Item item){
	       this.item=item;
	    }
	}

####使和谐结合起来的Activity

MainActivity在onCreate()中设置了fragment，在onResume()和onPause（）中注册和注销了event bus，并在onEventMainThread()
中处理了点击事件：

	package com.commonsware.android.hurl;

	import android.app.Activity;
	import android.content.Intent;
	import android.net.Uri;
	import android.os.Bundle;

	import de.greenrobot.event.EventBus;

	public class MainActivity extends Activity{
		@Override
		protected void onCreate(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);

			if(getFragmentManager().findFragmentById(android.R.id.content)==null){
				getFragmentManager().beginTransaction()
				                     add(android.R.id.content,new QuestionsFragment()).commit();
			}
		}

        @Override
        public void onResume(){
        	super.onResume();
        	EventBus.getDefault().register(this);
        }

        @Override
        public void onPause(){
        	EventBus.getDefault().unregister(this);
        	super.onPause();
        }

        public void onEventMainThread(QuestionClicked event){
        	startActivity(new Intent(Intent.ACTION_VIEW,Uri.parse(event.item.link)));
        }
	}

因此，MainActivity承担了一个使各种不同东西和谐结合起来的角色。QuestionsFragment是本地的控制器，处理由它的控件引发的直接事件。
MainActivity负责处理越过一个单独fragment的事件－在此例中，它启动了一个浏览器来看点击过的问题。

结果是展示问题的简单列表：
![wizard](/the_busy_coder's_guide/img/figure_266.png)

####Android带来了什么

Google增强了HttpUrlConnection来做更多的事情来帮助开发者。注意：

* 请求时自动使用Gzip压缩，添加适当的HTTP请求头来自动解压缩任意压缩过的返回(在Android 2.3添加的)
* 它使用了Server Name Indication来帮助一些https注解来共享单个IP地址
* API LEVEL 13(Android 4.0的时候)添加了一个java.net.ResponseCache基类的HttpResponseCache实现，设置之后就能显而易见地对你的HTTP请求进行混存。

也要，感谢一些我们会简短论述的第三方代码(OkHttp)，Android 4.4时 HttpUrlConnection也支持用于在SSL的网页内容发布加速的[SYDP协议](https://en.wikipedia.org/wiki/SPDY).


####StrictMode下的测试

在文件那章提及到的StrictMode，在主应用线程执行网络I／O的时候也会被报告。更重要的，在Android3.0及以上的设备，默认情况下，当你在主现场执行networkI／O的时候，Android应用会闪退并抛出
NetworkOnMainThreadException异常。

因此，使用StrictMode或合适的模拟器来确保你没有在主应用线程执行networkI／O是个好主意。

####那HttpClient呢？

Android也包含或者曾经包含一个几乎完全一样的 4.0.2beta版本的Apache Httpclient类库。许多开发者使用这个，
因为他们宁可要由这个类库提供的更丰富的API而不去管它的使用比java.net笨重。并且，说句实话，在Android 2.3之前
它更为稳定。

这有一些这个类库不再被Android2.3及以上版本推荐的原因：

* 核心Android小组更能够添加功能到java.net实现的同时而维持向后兼容性，因为它的API范围比较小。
* 之前Android中遇到的java.net实现的问题，大量的已经被修复过了。
* Apache HttpClient项目持续不断的演化它的API。这就意味着Android会继续落后并且落后的越来愈多于最新的ApacheHttpClient类库。因为
 Android坚持维护最好的向后兼容可能性，因此不能使用较新的但不同的HttpClient版本。
* Google官方在Android 5.1中废弃了这个API。
* Google官方在Android 6.0中移除了这个API。

如果有着使用HttpClent API的遗留代码，请考虑使用Apache的[给Android使用的HttpClient独立版本](https://hc.apache.org/httpcomponents-client-4.3.x/android-port.html)。

并且，如果你不能这么做，并且你使用给Android用的Gradle来构建的话（例如，你使用的是AndroidStudio的默认设置），你可以在android闭包中添加
'org.apache.http.legacy'来让你能访问Android的库存HttpClient API：
		
		android{
			useLibrary 'org.apache.http.legacy'
			//其它设置在这
		}








