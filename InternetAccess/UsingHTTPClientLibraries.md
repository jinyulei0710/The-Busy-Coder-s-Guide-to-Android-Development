###使用HTTP客户端类库

大部分时候，在多个主体部分，写网络访问代码是一种痛苦。

并不令人感到惊讶的是，有多种协助编写网络代码而设计的第三方类库。一些是设计成访问特定API的，例如这章之前内容提及到的。

但是，其它的是更加通用的，通过处理以下的事情使编写HTTP操作变的简单而设计的：

* 重试（例如，设备从WIFI切换到移动蜂窝数据时保持不断线）
* 线程（例如，在后台线程中为你处理网络工作）
* 众所周知格式的数据解析和封送处理（例如，JSON）

在这节中，我们会看到三个简化这个方式的类库:OkHttp,Retrofit,和Picasso。之后呢，我们会看下其它可能希望探讨的其它类库，包含Google自己的Volley HTTP 客户端 API。

####OKHttp

[OkHttp](http://square.github.io/okhttp/)使用了标准HttpUrlConnection的修改版本从而提供了许多性能上的提升。最值得注意的是它对SPDY的支持，一个Google赞助增强的HTTP版本，在传统的HTTP持久连接之上允许许多请求和响应在相同的socket连接上进行递送。许多Google API是由具备[SPDY](http://en.wikipedia.org/wiki/SPDY)功能的服务器提供的，并且SPDY也可以被其它的进行使用。

除此之外，OKHttp封装了常见的HTTP性能提升模式，例如GZIP压缩，响应缓存和连接池。它也对真实世界的连接问题有更多的了解，像错误配置的
代理服务器等等。

注意，一个OKHttp版本背后是Android 4.4及以上中标准的HttpUrlConnection实现－这是Android的SPDY支持的来源。

[Internet/OKHttp3](https://github.com/commonsguy/cw-omnibus/tree/master/Internet/OkHttp3)样例项目是之前在这章展示的Stack Overflow样例的一个拷贝。原本的样例使用HTTPURLConnection来下载Stack
Exchange 网站服务数据。这个修改的样例把HttpURLConnection换成了OkHttp。

首先，我们需要把OkHttp上的依赖添加到我们app/module的build.gradle文件中:
		
		dependencies {
        compile 'de.greenrobot:eventbus:2.4.0'
        compile 'com.google.code.gson:gson:2.3'
        compile 'com.squareup.okhttp3:okhttp:3.2.0'
       }

OkHttp提供了两个基本风格的HTTP API,同步的和异步的。使用一个同步的调用，调用会阻塞直到到HTTP I/O完成（或者，至少，头部下载好了）。使用一个异步的调用，
网络I/O的初始maichong是在一个后台线程处理的。一般的经验法则是:

* 如果你可以使用原始的HTTP响应，并且它是短的话，那就使用异步API，让你不用为你自己的线程烦恼
* 如果响应世界可能很长并且需要大量的读出工作（例如，解析），使用你自己的后台线程并使用同步API

在我们的例子中，我们需要使用Gson来解析JSON，并且第二种方式是更好的回答。它的副好处是把我们的Java改变
限制在LoadThread中：

		package com.commonsware.android.okhttp;

		import android.util.Log;
		import com.google.gson.Gson;
		import java.io.BufferedReader;
		import java.io.Reader;
		import de.greenrobot.event.EventBus;
		import okhttp3.OkHttpClient;
		import okhttp3.Request;
		import okhttp3.Response;

		class LoadThread extends Thread{
            static final String SO_URL=
             "https://api.stackexchange.com/2.1/questions?"
               + "order=desc&sort=creation&site=stackoverflow&tagged=android";
           
            @Override
            public void run(){
            	try{
				OkHttpClient client=new OkHttpClient();
				Request request=new Request.Builder().url(SO_URL).build();
				Response response=client.newCall(request).execute();

				if(response.isSuccessful()){
					Reader in=response.body().charStream();
					BufferedReader reader=new BufferedReader(in);
					SOQuestions questions=
					        new Gson().fromJson(reader,SOQestions.class);
					reader.close();
					EventBus.getDefault().post(new QuestionsLoadedEvent(questions))       
				}
				else{
					Log.e(getClass.getSimpleName(),response.toString());
				}
			}
			catch(Exception e){
				Log.e(getClass().getSimpleName(),"Exception parsing JSON",e);
			}
          }
		}


我们修改的run()方法创建了一个OkHttpClinet的实例。这是你执行HTTP请求的门户。

一个独立的请求，毫不意外的，是由一个Request对象表达的，通常使用的是一个Request.Builder。Build上的url()方法是你提供通过一个get请求获取的URL的地方。
Builder也有其它的方法，例如post()，它提供了请求的附加数据并把它转换进一个POST请求。[OKHttp秘诀页面](https://github.com/square/okhttp/wiki/Recipes)列出了一些常见情形的提纲。

为了实际执行同步请求，在OkHttpClient上调用newCall().execute()，传入Request给newCall()方法。
这个给了你一个Response对象，它的isSuccessful()应该返回true。false的话指示的是某种类型的问题，例如一个HTTP
404响应码。

有了一个成功的响应，你可以通过一个body()方法获取响应的主体.这个会返回一个ResponseBody,它提供了三个方法来获取
响应主体本身：

* string()以字符串返回整个响应，这个方法只合适于简短的文本响应
* byteStream()将响应体的原始字节以一个InputStream返回
* charStream(),除了它的名字外，在响应特征上返回一个Reader，这个Reader考虑到了响应体的字符编码(例如,UTF-8)

这里，我们使用charstream()获取了一个Reader,然后我们把这个Reader封装进一个BufferedReader.run()方法的其余部分
跟原本的是几乎一样的，要求Gson解析响应体并在应用主线程上发起一个QuestionsLoadedEvent来获取问题到我们的fragment。

####Retrofit

当使用HTTP请求的时候，多次的我们的需求是相当简单的，只是从一些网站服务取回一些JSON(或其它例如XML的机构数据)，或者上传一些JSON到网站服务。

[Retrofit](http://square.github.io/retrofit/)是为了简化这个过程而设计的，为我们处理数据解析和封装，HTTP操作以及（可选的）后台线程。我们被留下
一个相当自然的Java API来进行 发送/接受 Java对象 到／从 网站服务。

Retrofit通过灵巧地使用注解，反射以及OkHttp本身来完成这个工作。

为了展示Retrofit,让我们查看下[HTTP/Retrofit](https://github.com/jinyulei0710/cw-omnibus/tree/master/HTTP/Retrofit)这个样例项目。这个项目是之前的一个拷贝，这次试用的
是Retrofit来取回我们的StackFlow问题。

注意StackOverflow正好使用JSON作为它的数据格式，和Retrofit配合的非常好，因为JSON是它默认的数据格式。
但是，你也可以提供你自己的转换逻辑，把数据转化到/从其它格式，例如XML或[Protocol Buffers](https://developers.google.com/protocol-buffers/)。

####下载和安装Retrofit

Retrofit可以以一个小JAR包的形势从前面提到的[Retrofit网站](http://square.github.io/retrofit/)上下载。在Android Studio中则需要加入如下依赖：

		dependencies {
            compile 'com.squareup.retrofit2:retrofit:2.0.0'
            compile 'com.squareup.retrofit2:converter-gson:2.0.0'
            compile 'org.greenrobot:eventbus:3.0.0'
         }

####创建你的服务接口

第一件我们需要告诉Retrofit更多的是我们的JSON从哪来。为了这么做，我们需要使用一些特定的由Retrofit提供的注解和描述来创建一个Java接口

* 我们想要执行的HTTP操作
* 路径（如果有的话请求参数）来应用一个HTTP操作
* 配置HTTP操作的预先请求数据，例如用于REST样式API的路径的动态部分，附加到URL的补充参数
* HTTP响应应该倒入到什么对象

例如，让我们看下StackOverflowInterface，我们的接口用于请求Stack Exchange的API来从StackOverflow获取问题：

     package com.commonsware.android.retrofit;

		import retrofit2.Call;
		import retrofit2.http.GET;
		import retrofit2.http.Query;

		public interface StackOverflowInterface {
  			@GET("/2.1/questions?order=desc&sort=creation&site=stackoverflow")
 			 Call<SOQuestions> questions(@Query("tagged") String tags);
		}

接口中的每个方法应该有一个识别要执行HTTP操作的注解，例如@GET或@POST。注解的参数是请求的路径以及所有固定的
请求参数。在我们的例子中，我们使用了由Stack Exchange文档提供的取得问题的路径(/2.1/questions),外加一些固定的请求参数：

* order用于结果是应该生序还是降序
* sort指示问题是如何排序的，例如creation就是按创建事件排序
* site指示我们请求的是哪个Stack Exchange的站点(例如，stackoverflow)

方法名则是随意的。

如果你有额外的动态变化的请求参数，你可以在String参数上使用@	Query注解让它们添加到URL的末尾。
在我们的例中，tagged请求参数会和我们questions()方法中的参数一起添加。

同样的，你可以使用{name}占位符用于path的部分，并且在运行时通过@Path-注解参数替换方法中的参数。


为了获取返回的结果，并指示结果的数据类型，你有着两个选择，同步和异步，但是在2.0中是同一个声明方式。


说来古怪，我们永远不会自己创建StackOverflowInterface的实现。作为替代，Retrofit为我们生成了一个，这个代码实现了我们要求的行为。

####创建Retrofit

为了使用这个生成的StackOverflowInterface，并实际执行这些操作，我们需要创建一个Retrofit实例。通常，我们会通过Retrofit.Builder来配置你
想要完成的。

你要提供给Retrofit.Builder的最大的事情是这些HTTP操作绑定的服务。调用baseUrl()允许你详细说明URL的模式，主机以及端口。调用addConverterFactory()
允许你设置数据转化所使用的格式。例如此例中，我们连接的是https://api.stackexchange.com服务，使用的是Gson，所以我们有：


		Retrofit retrofit=new Retrofit.Builder()
            .baseUrl("https://api.stackexchange.com")
            .addConverterFactory(GsonConverterFactory.create())
            .build();

当你完成Retrofit.Builder的配置，调用build()来获取结果retrofit。


####发起请求

有了配置好的Retrofit，你可以通过调用create()方法取回一个你的API接口的实现：
     
         StackOverflowInterface so=
            retrofit.create(StackOverflowInterface.class);

你可以无差异地像其他Java对象一样使用结果的接口类型的对象，尽管你从没有自己写这个接口的实现。

在我们的例子中，我们可以调用questions()方法，并提供给它我们想要的最近问题的tag(或多个tag)：
        
         Call<SOQuestions> call=so.questions("android");


然后执行call的入队操作，成功时，更新listview的adapter,失败时，toast错误信息，并打印错误日志。

         call.enqueue(new Callback<SOQuestions>() {
          @Override
          public void onResponse(Call<SOQuestions> call, Response<SOQuestions> response) {
             setListAdapter(new ItemsAdapter(response.body().items));

          }

           @Override
            public void onFailure(Call<SOQuestions> call, Throwable t) {
              Toast.makeText(getActivity(), t.getMessage(),
                Toast.LENGTH_LONG).show();
              Log.e(getClass().getSimpleName(),
                "Exception from Retrofit request to StackOverflow", t);
         }
      });                

####Picasso

有时候，你想要下载的东西不是JSON,或XML或任意类型的结构数据。

有时候，它是一张图片。

例如，Stack Overflow用户有头像。在我们的样例应用中，展示问这个问题的用户的头像可能比较好。

[Picasso](http://square.github.io/picasso/)是一个来自Square为帮助异步加载图片而设计的类库，不管图片是来自HTTP请求，本地文件，内容提供者，等。
除了进行异步加载之外，Picasso简化了很多在这些图片上的操作，例如：

* 结果缓存(对HTTP请求在磁盘上可选)
* 当图片在加载中是显示占位图片，加载图片出现问题的时候显示错误图片（例如，非法的URL）
* 改变图片，例如调整图片大小或者把它剪切来适应一定量的空间
* 把图片直接加载进你选择的一个ImageView，就连ImageView回收的情况也处理了。

[HTTP/Picasso]样例程序扩展了Retrofit那个样例来加载问问题那个人的头像，把它和问题的标题一起展示在ListView中。

####下载和安装Picasso

Picasso可以以小JAR包的形式从[之前提到的网站](http://square.github.io/picasso/)下载。Android Studio用户可以只添加一个用于com.squareup.picasso：picasso:...
的依赖，...代表版本号。并且它会拉取其它被Picasso需要的所有其它依赖。

####更新模型

我们原本的数据模型不包含拥有者的信息。因此，我们需要参加我们的数据模型，从而让Retrofit从Stack Exchange Json拉取信息并使之对我们可用。

为此，我们现在有了一个Owner类，含有我们所需的拥有者一个信息：avatar的URL(又名，“profile image):

		package com.commonsware.android.picasso;

		import com.google.gson.annotations.SerializedName;

		public class Owner{
			@SerializedName("profile_image")String profileImage;
		}

Stack Exchange API中用于这个的JSON键是profile_image,并且下划线不是在Java成员中隔开单词的常见做法。Java作为代表的是驼峰。
Retrofit的默认行为需要我们把我们的数据成员命名为profile_image以映射JSON。

但是，表象之下，Retrofit使用的是Google的Gson来从JSON映射到对象。Gson支持一个@SerializedName注解，来指示JSON键使用这个
数据成员。这允许我吗给予数据成员profileImage更自然的名词，通过使用@SerializedName("profile_image")来教导Gson如何
填充它的属性。

我们的Item类现在有了一个名叫owner的Owner，因为owner数据是items的JSON对象中的owner键：
		
		package com.commonsware.android.picasso;

		public class Item{
			String title;
			Owner owner;
			String link;

			@Override
			public String toString(){
				return(title);
			}
		}

这两个改变对于Retrofit给予我们下载图片的URL是足够了。

####请求图片

使用Picasso是非常简单的，因为它提供了一个优美自然的接口允许我们在单个Java声明中
设置一个请求。

这个声明一调用Picasso类的静态with()方法开始，在这个方法中我们提供了一个Context(例如我们的activity)来给Picasso使用。
声明以into()调用结尾，来指示Picasso记载到哪个ImageView。在这些调用之前，我们可以链接其它调用，因为with()和大都数在Picasso上的其它方法
会返回Picasso对象本身。

所以，我们可以做一些像这样的事：

		Picasso.with(getActivity()).load(item.owner.profileImage)
		       .fit().centerCrop()
		       .placeholder(R.drawable.owner_placeholder)
		       .error(R.drawable.owner_error).into(icon);

这里，我们:

* 指示我们想要以某一URL记载图片，标识为Item中Owner中的profileImage
* 告诉图片我们要图片fit()我们的目标ImageView
* 指定图片调整大小时应该使用centercrop规则
* 指定某一图片作为图片记载时的占位图片
* 指定某一图片作为图片加载错误的时图片

就是这样。Picasso启动，下载图片，当准备好的时候把图片注入ImageView。

####故事的剩余部分

Picasso这一点代码在我们新ItemAdapter上的getView()方法中:

		@Override
		public View getView(int position,View convertView,ViewGroup parent){
			View row=super.getView(position,convertView,parent);
			Item item=getItem(position);
			ImageView icon=(ImageView)row.findViewById(R.id.icon);

			Picasso.with(getActivity()).load(item.owner.profileImage)
			.fit().centerCrop()
			.placeholder(R.drawable.owner_profileImage)
			.error(R.drawable.owner_error).into(icon);

			TextView title=(TextView)row.findViewById(R.id.title);

			title.setText(Html.fromHtml(getItem(positoin).title));

			return(row);
		}

我们创建了我们自己的行布局(res/layout/row.xml)，包含了一个ImageView和一个TextView。
我们有ArrayAdapter inflate或recycle我们的行，取回这个布局的Item,从行中取出ImageView，
使用Picasso开始下载真正的图片，把HTML实体文本填充进TextView，然后返回我们更新的行。当返回行的
时候，Picasso已经加载好了占位图片，它是在我们下载真实图片时用户最初看到的。

结果是现在在每个我们问题标题的旁边有了图标:

![wizard](/the_busy_coder's_guide/img/figure_267.png)


####Volley

####TODO



