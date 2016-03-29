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





